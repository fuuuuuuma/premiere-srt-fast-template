# スキルテンプレート

壁打ちの回答を元に、以下の {{変数}} を埋めてスキルファイルを生成してください。
生成したファイルは `~/.claude/commands/{{COMMAND_NAME}}.md` に保存する。

## 変数一覧

| 変数 | 対応する質問 |
|---|---|
| `{{COMMAND_NAME}}` | Q9 コマンド名 |
| `{{PROJECT_ROOT}}` | このリポジトリの絶対パス（`pwd` で取得。ユーザーには聞かない） |
| `{{INPUT_DIR}}` | Q7 音声/動画の読み込み場所 |
| `{{OUTPUT_DIR}}` | Q8 SRT出力先 |
| `{{USE_XML}}` | Q10 カット点同期XMLを使うか |

---

## 生成するスキルファイルの内容

````markdown
---
description: WAV/動画音声を単一パスGPU転写（mlx-whisper）した後、fulltextを意味区切りでN分割しLLM改行だけを並列化してSRT字幕を高速生成する。日本語トーク動画用。
---

# WAV → SRT 高速生成 (/{{COMMAND_NAME}})

## 使い方

```
/{{COMMAND_NAME}} /path/to/audio_or_video
```

パート数を変えたい場合は `prepare_text_parts.py --n <数>`（既定0=文字数から自動、上限10）。
{{#USE_XML}}
カット点同期 XML がある場合は `--xml "<XML>"` を付けると manifest に記録され、
Step 4 の `--from-text` に引き渡される。
{{/USE_XML}}

## 実行手順

### Step 0: チャンネル設定の確認

`{{PROJECT_ROOT}}/config/channel_profile.md` を読む。**存在しない場合のみ**、
チャンネル名・目標文字数・固有名詞・半角スペースの好みを質問してから
`config/channel_profile.example.md` の書式で作成する（通常は導入時の壁打ちで
既に作成済みのはずなので、この質問が発生するのは再セットアップ時のみ）。

### Step 1: 入力確認

引数の音声/動画の絶対パスを確認する（存在しなければユーザーに確認）。パスは【】や空白を含み得るので
以降ダブルクオートで囲む。

### Step 2: 前処理（bash・単一パス転写＋テキスト分割・agentゼロ）

```
python3 "{{PROJECT_ROOT}}/scripts/chunk_tools/prepare_text_parts.py" "<入力の絶対パス>"
```

**Bash タイムアウト: 600000ms（10分）必須**（GPU転写は音声長の約1/8だが長尺に備える。
mlx-whisper が無い環境では自動的に CPU 並列転写にフォールバックするため、より時間がかかる場合がある）。
10分超が見込まれる長尺は、`run_in_background: true` で実行し完了通知を待ってから次へ進む。

標準出力の `MANIFEST: <path>` が `<stem>.parts.json` の絶対パス。

### Step 3: manifest を読み、改行エージェントをN体並列起動（直Agent・1メッセージ）

Read で `<stem>.parts.json` を取得し、**parts の数だけ Agent を同一メッセージで起動**する
（`model: sonnet` 必須・`subagent_type: general-purpose`）。各エージェントのプロンプトは
次のテンプレート（`{...}` を manifest の値で置換）:

```
あなたは日本語トーク動画SRTテロップの「意味区切り改行」担当エージェントです。
転写済み全文を{n}分割したパート {idx}/{n} を担当します。あなたの仕事は改行だけです。

## Step 1: ルール正典を読む（必須・全ルール厳守）
Read: {{PROJECT_ROOT}}/references/srt_runtime_rules.md
Read: {{PROJECT_ROOT}}/config/channel_profile.md
（存在すれば。無ければスキップしてよい。存在すればそこに書かれたチャンネル固有の表記・目標値を優先する）

## Step 2: 担当パート全文を読む
Read: {parts[i].path}

## Step 3: 意味区切り改行 → Write
パート全文をルール正典に従って意味の区切りで改行し（各行=1テロップ）、
{parts[i].lines_out} に Write する。

【鉄則】冒頭・末尾が中途半端に見えても削除・要約・言い換えをせず全文をカバー
（削除して良いのはルール正典の削除規定該当箇所のみ）。SRT・タイムコード・行番号は
書かない。25字を超えそうな行は積極分割ルールで割る（目標: 25字超1%未満・平均14字前後、
{{PROJECT_ROOT}}/config/channel_profile.md に別の目標値があればそちらを優先）。
書き終えたら再読・再検証はせず即終了する。

最終応答は「LINES=<非空行数>」だけを返す。
```

- **「書き終えたら再読・再検証せず即終了」は速度の要**（これが無いと自己検証の
  脇道に入り3〜4倍遅くなる）。
- 完了は `<task-notification>` で通知される。全パート完了まで待つ。

### Step 4: 組み立て＋QA（bash直・エージェント不使用）

```bash
cd "{{OUTPUT_DIR}}" && python3 -c "
from pathlib import Path
import json
m = json.loads(Path('<stem>.parts.json').read_text())
lines = []
for p in m['parts']:
    lines += [l.strip() for l in Path(p['lines_out']).read_text().splitlines() if l.strip()]
Path(m['lines_out']).write_text('\n'.join(lines)+'\n')
print(len(lines), 'lines')
" && SRT_QA_JSON=1 python3 "{{PROJECT_ROOT}}/scripts/whisper_to_srt.py" \
  --from-text "<stem>.fast.lines.txt" --segments "<stem>.segments.json" -o "<stem>.fast.srt"
```

{{#USE_XML}}
manifest の `xml` が null でなければ `--from-text` コマンドに `--xml "<manifestのxml>"` を追加する。
{{/USE_XML}}
標準出力末尾の `QA_JSON: {...}` を読む。

### Step 5: QA修復（メインループが直接・最大2周）

`over25 > max(1, total×1%)` または `head_ng > 0` の場合のみ:
`QA_JSON` の `over25_items` / `head_ng_items` の各テキストは `<stem>.fast.lines.txt` の1行に
一致する。**該当行だけ**を Edit で修正し（25字超→ルール正典の積極分割で2行に / 文頭NG→
区切りを前行側へ移動。行の削除・要約・語の追加はしない）、Step 4 の `--from-text` コマンド
だけを再実行（約1秒）。2周やっても改善しなければ打ち切って現状を報告する。

### Step 6: 掃除と完了報告

```bash
rm -f "{{OUTPUT_DIR}}/<stem>".part*.txt "{{OUTPUT_DIR}}/<stem>".part*.lines.txt
```

1. 最終 SRT の絶対パス（`<stem>.fast.srt`）
2. 統計表（エントリ数・平均文字数・25字超・4字未満・最大空白秒・QA修復周回数）
3. 所要時間（prepare / 並列改行 / 組立・修復）
4. 「Premiere Pro にインポートできます」

## 入出力パス（このリポジトリ固有の設定値）

- 音声/動画の読み込み場所（既定）: `{{INPUT_DIR}}`
- SRT出力先: `{{OUTPUT_DIR}}`
- 新しい固有名詞誤認識を見つけたら `{{PROJECT_ROOT}}/config/corrections.local.json` に
  追記し、上記 --from-text を再実行すれば表示行にも即反映される。

## CPUフォールバックについて

Apple Silicon Mac で `mlx-whisper` が使えない環境（Windows/Linux/Intel Mac、または
`SRT_WHISPER_ENGINE=cpu` 指定時）は、Step 2 の転写が自動的に
`scripts/transcribe_parallel.py --jobs 3`（音声3分割・並列CPU転写・境界復元込み）に
切り替わる。この経路が使う `scripts/chunk_tools/setup_chunks.py` は BGM/環境音が多い素材で
分割点検出の `--noise` 調整（既定 -30dB、効きが悪ければ -40 等）が必要な場合がある。
````
