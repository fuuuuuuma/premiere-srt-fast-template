# WAV/動画 → SRT 高速生成 — セットアップウィザード

## 起動判定（まずここを確認）

- カレントディレクトリに `.template/questions.md` がローカルに存在し読める
  → **既にこのリポジトリがclone済み**。Step B へ進む
- 存在しない・読めない（＝チャット越しにこのファイルを読んでいるだけで、まだcloneされていない）
  → Step A へ進む

## Step A: 未clone（GitHubのURLを渡されてこのファイルを読んでいる場合）

**重要**: このファイルの内容を WebFetch で読んでいる場合、`.template/questions.md` 等の
正確な原文は WebFetch では取得できない（要約されてしまい壁打ちの文言が変質する）。
必ず先にリポジトリを一時ディレクトリへ clone し、以降は Read でローカルファイルを読むこと。

1. `~/.claude/CLAUDE.md` が存在すれば読み込み、文体・スタイルを把握する
2. まず OS の一時ディレクトリ配下にこのリポジトリを clone する（保存先はまだ聞かない）:
   ```bash
   TMPDIR_SETUP="$(mktemp -d)/premiere-srt-fast-template"
   git clone https://github.com/fuuuuuuma/premiere-srt-fast-template.git "$TMPDIR_SETUP"
   ```
3. clone した `$TMPDIR_SETUP/.template/questions.md` を **Read** で読み、Q1〜Q3 を
   **1問ずつ** ユーザーに聞く（まとめて聞かない）
4. Q3（保存先）の回答が絶対パスで得られたら、親ディレクトリを作ってから一時ディレクトリを移動する:
   ```bash
   mkdir -p "$(dirname "<Q3の絶対パス>")"
   mv "$TMPDIR_SETUP" "<Q3の絶対パス>"
   ```
5. 移動後の絶対パスを `{{PROJECT_ROOT}}` として Step B へ進む（Q1・Q2は聞き直さない）

## Step B: 生成（ローカルにこのリポジトリがある状態）

1. `{{PROJECT_ROOT}}` を確定する（Step Aから来た場合は移動後の絶対パス。直接この状態から
   始まった場合は `pwd` でこのリポジトリの絶対パスを取得し、まだ聞いていなければ
   `.template/questions.md` の Q1・Q2 を質問する）
2. `.template/structure.md` を元に `config/channel_profile.md`（Q1の回答）と
   `config/corrections.local.json`（Q2で回答があれば）を作成する
3. `.template/skill-template.md` の `{{PROJECT_ROOT}}` を実際の絶対パスに置換したものを
   `~/.claude/commands/srt-fast.md` に保存する
4. `ffmpeg -version` と `python3 -c "import faster_whisper"` を実行し、未インストールなら
   `pip3 install --user faster-whisper` / `brew install ffmpeg` を案内する
   （Apple Silicon Mac の場合は `pip3 install --user mlx-whisper` も併せて案内し、
   入っていれば自動でGPU転写が使われる旨を伝える）
5. `/srt-fast <音声ファイルの絶対パス>` で使い始められる旨を伝えて終了する

## 注意

- 質問は必ず1問ずつ。まとめて聞かない（Step Aは3問、Step Bのみから始まる場合は2問）
- テンプレート原文（questions.md / structure.md / skill-template.md）は必ず **Read で
  ローカルファイルとして** 読む。WebFetch は使わない（要約されて内容が変質するため）
- `scripts/` `references/` は既に同梱済みのファイルなので変更しない
- コマンド名は常に `srt-fast` で固定。出力先は常に `{{PROJECT_ROOT}}/output/srt/<ファイル名>/`
  （`scripts/chunk_tools/prepare_text_parts.py` が自動決定するため、これらは聞かない）
- セットアップ後にフォルダを移動したくなった場合は、手動でフォルダごと移動してから
  そのディレクトリで再度セットアップを実行すればよい（Step B が新しい `pwd` を
  `{{PROJECT_ROOT}}` として `~/.claude/commands/srt-fast.md` を作り直す）
