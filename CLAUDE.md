# WAV/動画 → SRT 高速生成 — セットアップウィザード

このリポジトリを読み込んだら、以下を自動で実行してください。

## 起動手順

1. `~/.claude/CLAUDE.md` が存在すれば読み込み、文体・スタイルを把握する
2. `.template/questions.md` を読む
3. 質問を **1問ずつ** ユーザーに聞く（まとめて聞かない）
4. 全回答が揃ったら `.template/structure.md` を元に `config/channel_profile.md` と
   （固有名詞の回答があれば）`config/corrections.local.json` を作成する
5. `.template/skill-template.md` を元にスキルファイルを生成し、
   `~/.claude/commands/{{COMMAND_NAME}}.md` に保存する

## 注意

- 質問は必ず1問ずつ。まとめて聞かない
- ユーザーが答えを迷っていたらデフォルト値を提示して選ばせる
- `{{PROJECT_ROOT}}` は `pwd` で取得したこのリポジトリの絶対パス。ユーザーには聞かない
- `scripts/` `references/` は既に同梱済みのファイルなので変更しない。壁打ちで作るのは
  `config/channel_profile.md` と `config/corrections.local.json` の2ファイルだけ
- ffmpeg / mlx-whisper / faster-whisper が未インストールの場合はインストール手順も出力する
- 生成が終わったら、`/{{COMMAND_NAME}} <音声ファイルの絶対パス>` で使い始められる旨を伝える
