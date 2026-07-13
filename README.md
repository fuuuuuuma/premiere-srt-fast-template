# premiere-srt-fast-template

WAV/動画音声から日本語トーク動画用のSRT字幕を高速生成する `/srt-fast` の **セットアップウィザード** テンプレートです。

## 使い方（Claude Code）

ターミナルで `git clone` する必要はありません。Claude Code に次のメッセージを渡すだけです。

> `https://github.com/fuuuuuuma/premiere-srt-fast-template` の CLAUDE.md に従ってセットアップして

壁打ち（3問）が始まります。答え終わると保存先フォルダに自動でセットアップが作られ、
`/srt-fast` がすぐ使えるようになります。

## 壁打ちで聞かれること

1. テロップ1行の目標文字数（デフォルト: 平均14字前後・25字超1%未満）
2. このチャンネルでよく出る固有名詞・専門用語・人名（Whisperが誤変換しそうなもの）
3. セットアップの保存先フォルダ（絶対パス）

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

- `~/.claude/commands/srt-fast.md` — Claude Code スキルファイル
- `config/channel_profile.md` — 目標文字数の設定
- `config/corrections.local.json` — このチャンネル固有の固有名詞・言い間違い辞書

## 必要な環境

- Claude Code（CLI・デスクトップアプリいずれも可）
- Python 3.9 以降
- `faster-whisper`（`mlx-whisper` は Apple Silicon Mac のみ・任意だが高速）
- `ffmpeg`
- 未インストールの場合はウィザードがセットアップ手順を案内する

## 学習サイクル

1. `/srt-fast` で生成
2. Premiere Pro でテロップを手動微調整
3. 気づいた固有名詞の言い間違いを `config/corrections.local.json` に追記
4. 次回の生成から自動反映される

## 関連ツール

生成したSRTをPremiere Proに読み込んでテロップ（グラフィッククリップ）化した後、
イン点・アウト点を基準トラックへ自動スナップしたい場合は
[Caption Align](https://github.com/fuuuuuuma/caption-align-plugin?tab=readme-ov-file)
（Premiere Pro UXPプラグイン）が使える。
