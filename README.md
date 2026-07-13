# premiere-srt-fast-template

WAV/動画音声から日本語トーク動画用のSRT字幕を高速生成する `/srt-fast` の **セットアップウィザード** テンプレートです。

## 使い方

```bash
git clone https://github.com/fuuuuuuma/premiere-srt-fast-template.git
cd premiere-srt-fast-template
claude
```

Claude Code を起動すると壁打ちウィザードが始まります。11項目の質問に答えるだけで、
自分のチャンネルに合わせた `/srt-fast` スキルが `~/.claude/commands/` に生成されます。

## 何をしてくれるか

- Whisper（`mlx-whisper` / `faster-whisper`）で音声を転写
- 転写結果を意味の区切りでLLMが改行（複数エージェント並列・高速）
- 文字数超過・文頭フラグメントを自動検出して修復
- Premiere Pro にそのままインポートできるSRTを出力

## 同梱されているもの（変更不要）

- `scripts/` — 転写・組み立てスクリプト（`whisper_to_srt.py` 等）。GPU（mlx-whisper）が
  無い環境では自動的にCPU並列転写にフォールバックする
- `references/srt_runtime_rules.md` — テロップ改行の実行時ルール正典（意味区切り・
  固有名詞・目標文字数の判断基準）

## 壁打ちで生成されるもの

- `~/.claude/commands/{{コマンド名}}.md` — Claude Code スキルファイル
- `config/channel_profile.md` — チャンネル名・目標文字数・スペース使いの好み
- `config/corrections.local.json` — このチャンネル固有の固有名詞・言い間違い辞書

## 必要な環境

- Claude Code CLI
- Python 3.9 以降
- `faster-whisper`（`mlx-whisper` は Apple Silicon Mac のみ・任意だが高速）
- `ffmpeg`
- 未インストールの場合はウィザードがセットアップ手順を生成する

## 学習サイクル

1. `/srt-fast` で生成
2. Premiere Pro でテロップを手動微調整
3. 気づいた固有名詞の言い間違いを `config/corrections.local.json` に追記
4. 次回の生成から自動反映される
