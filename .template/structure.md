# フォルダ構成テンプレート

このリポジトリの `scripts/` `references/` は既に同梱済み（変更不要）。壁打ちの回答をもとに
`config/` 配下の2ファイルだけを新規作成する。

```
{{PROJECT_NAME}}/                       ← このリポジトリ（clone先）
├── scripts/                            ← 同梱済み・変更不要
│   ├── whisper_to_srt.py
│   ├── transcribe_parallel.py          ← CPUフォールバック経路
│   └── chunk_tools/
│       ├── prepare_text_parts.py
│       ├── setup_chunks.py
│       ├── whisper_chunk.py
│       └── merge_segments.py
├── references/
│   └── srt_runtime_rules.md            ← 同梱済み・改行ルール正典（変更不要）
├── config/
│   ├── channel_profile.md              ← ここで新規作成（Q1〜Q4の回答）
│   └── corrections.local.json          ← ここで新規作成（Q3の回答があれば）
├── input/                              ← Q7の書き出し場所からファイルをここにコピーして使う
└── output/
    └── srt/                           ← Q8の保存先
```

## `config/channel_profile.md` の内容（`config/channel_profile.example.md` の書式に従う）

```markdown
# チャンネル設定

## チャンネル名
{{CHANNEL_NAME}}

## テロップ目標
- 平均文字数の目安: {{TARGET_CHARS}}
- 25字超の許容率: {{OVER25_RATIO}}

## 半角スペースの好み
{{SPACE_STYLE}}

## 固有名詞・言い間違い辞書
実際の辞書は `config/corrections.local.json` に持つ（このファイルには方針だけメモする）。
```

## `config/corrections.local.json` の内容（Q3で回答があった場合のみ作成）

`config/corrections.example.json` の書式に従い、Q3で挙がったペアを
`{"誤認識文字列": "正規表記"}` 形式で書く。回答が無ければこのファイルは作らない
（`scripts/whisper_to_srt.py` は無くてもエラーにならず、汎用ルールだけで動く）。

## ffmpeg 未インストールの場合（Q11 が「いいえ」）

```bash
brew install ffmpeg
ffmpeg -version
```

## Whisperライブラリ未インストールの場合（Q6 が「いいえ」）

```bash
# Apple Silicon Mac（Q5ではい）の場合
pip3 install --user mlx-whisper faster-whisper

# それ以外（Q5いいえ・不明）の場合
pip3 install --user faster-whisper
```
