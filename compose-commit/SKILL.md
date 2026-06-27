---
name: compose-commit
description: >
  git コマンドで変更ファイルを解析し、論理的な単位で適切な粒度のコミットを作成する。
  ステージ済みまたは未ステージの git 変更が存在し、グループ化してコミットする必要がある場合に使用する。
allowed-tools:
  - Bash(git add *)
  - Bash(git branch *)
  - Bash(git commit *)
  - Bash(git config *)
  - Bash(git diff *)
  - Bash(git log *)
  - Bash(git stash *)
  - Bash(git status *)
  - Bash(git switch *)
  - Glob
  - Grep
  - Read
  - AskUserQuestion
---

# Git コミットの作成

変更ファイルを解析して関連する変更をグループ化し、適切な粒度でコミットを作成する。

## クイックスタート

1. ステージ済みまたは未ステージの変更がある状態で `/compose-commit` を実行するか「コミットして」と伝える
2. スキルが git status、ブランチ、最近のログを確認してコンテキストを把握する
3. 変更を解析して意図と変更タイプを推定する
4. コミット計画（グループとメッセージ）を提示する。確認または調整する
5. グループごとにコミットを実行し、最終的なログサマリーを表示する

## ステップ

### ステップ 1: 現在の状態を把握する

変更内容とプロジェクトのコミットスタイルを確認する。これらのコマンドは独立しているので、すべて並行して実行する:

```bash
git branch --show-current
git config --get commit.gpgsign
git status
git diff --stat HEAD
git log --oneline -10
git stash list
```

**GPG 署名が有効な場合（`commit.gpgsign=true`）、フラグを立てる。**
この時点ではユーザーへの確認は不要。ステップ 4 の承認後、ステップ 5 の実行前に確認する。

**`git stash list` にエントリがある場合**、続行前にユーザーへ通知する:

```
注意: N 件のスタッシュがあります。今回のコミットから意図的に除外されていることを確認してください。
```

処理はブロックせず、情報を表示して続行する。

**main または master ブランチにいる場合は必ずユーザーに確認する。**
現在のブランチが `main` または `master` の場合、コミット前に AskUserQuestion で確認する:

```javascript
AskUserQuestion({
  questions: [{
    question: "現在 main ブランチにいます。どのように進めますか？",
    header: "ブランチの確認",
    options: [
      { label: "作業ブランチを作成する", description: "コミット前に新しいブランチを作成する" },
      { label: "main にコミットする",     description: "現在のブランチに直接コミットする" }
    ],
    multiSelect: false
  }]
})
```

「作業ブランチを作成する」が選択された場合、続けてブランチ名をテキスト質問で尋ね、`git switch -c <branch>` を実行してから続行する。

すでにステージ済みファイルがある場合、続行前に AskUserQuestion でユーザーの意図を確認する:

```javascript
AskUserQuestion({
  questions: [{
    question: "すでにステージ済みのファイルがあります。どのように進めますか？",
    header: "ステージ済みファイル",
    options: [
      { label: "すべての変更を解析する",       description: "未ステージの変更も含めて解析する（ステップ 2 へ）" },
      { label: "このままコミットする",           description: "ステージ済みファイルのみをコミットする（ステップ 2 をスキップ）" }
    ],
    multiSelect: false
  }]
})
```

「すべての変更を解析する」が選択された場合、通常通りステップ 2 へ進む。

「このままコミットする」が選択された場合、`git diff --cached` でステージ済みの内容を確認し、すべてのステージ済みファイルを 1 つのコミットグループとして扱い、ステップ 4 へスキップする。

### ステップ 2: 変更を読んで意図を把握する

ファイルごとではなく、1 つのコマンドですべての差分を取得する:

```bash
git diff HEAD
```

出力を解析して各変更の意図を把握する。以下の観点から分析する:

- **変更の種類**: 機能追加 / バグ修正 / リファクタリング / テスト追加 / 設定変更 / ドキュメント更新
- **変更の目的**: なぜ変更したか（変更をまとめる軸となる）
- **ファイル間の関係**: 一緒に変更すると意味をなす組み合わせを探す

追跡されていない新規ファイルは `git diff HEAD` に含まれないため、Read でその内容を確認する。

### ステップ 3: 変更をグループ化してコミット計画を作成する

以下の原則に従って変更をグループ化する:

**まとめてよい変更**
- 同じ機能や目的に属するソースファイルとテスト
- 同じバグ修正のための一連のファイル
- 同じリファクタリングの対象ファイル

**別々のコミットにすべき変更**
- 目的が異なる（機能追加 vs バグ修正 vs リファクタリングなど）
- 互いに独立している（どちらも相手に依存していない）
- 影響範囲が異なる（アプリケーションコード / 設定 / ドキュメント / CI など）

変更した 1 ファイルに複数の意図が混在している場合、`git add -p` による部分ステージングを検討する。

### ステップ 4: 計画をユーザーに提示して確認を取る

**必ず実行前に確認する。** まず提案する計画をテキストで出力する:

```
提案するコミット（計 N 件）:

コミット 1: feat(auth): JWT ベースの認証を実装する
  ファイル:
    - src/auth/jwt.ts（新規）
    - src/auth/middleware.ts（変更）
    - tests/auth/jwt.test.ts（新規）

コミット 2: chore: ESLint 設定を更新する
  ファイル:
    - .eslintrc.json（変更）
```

続けて AskUserQuestion を使用する:

```javascript
AskUserQuestion({
  questions: [{
    question: "このコミット計画で進めますか？",
    header: "コミット計画",
    options: [
      { label: "進める",   description: "提案通りにコミットを実行する" },
      { label: "修正する", description: "グループ分けやコミットメッセージを調整する" },
      { label: "キャンセル", description: "コミットせずに中断する" }
    ],
    multiSelect: false
  }]
})
```

「修正する」が選択された場合、具体的な変更内容をテキスト質問で尋ね、計画を調整してステップ 4 の最初から再提示する。

「キャンセル」が選択された場合、即座に停止する。

### ステップ 5: グループごとにコミットを実行する

承認されたら、GPG が設定されていない場合はグループ順にコミットを作成する。

GPG フラグが立っている場合、コミット前に AskUserQuestion で確認する:

```javascript
AskUserQuestion({
  questions: [{
    question: "GPG 署名が有効です。Claude Code はパスフレーズを入力できないため、このセッションのコミットには --no-gpg-sign を使用します。よろしいですか？",
    header: "GPG 署名",
    options: [
      { label: "自分で実行する",               description: "ターミナルで手動実行するコマンドを出力する" },
      { label: "--no-gpg-sign でコミットする", description: "このセッションのみ GPG 署名をスキップする" },
      { label: "キャンセル",                   description: "コミットせずに中断する" }
    ],
    multiSelect: false
  }]
})
```

「キャンセル」が選択された場合、何も出力せずに停止する。

「自分で実行する」が選択された場合、コピー＆ペーストで実行できるコマンドとしてコミット計画を出力して停止する。heredoc の解釈の問題を避けるため、**`git add` と `git commit` コマンドは別々のブロックに分ける**。メッセージは（heredoc を使わず）引用符で囲んだ文字列を使用する:

````
GPG 署名が必要です。ターミナルから以下のコマンドを実行してください:

---

### コミット 1: `feat(auth): JWT ベースの認証を実装する`

```bash
git add src/auth/jwt.ts src/auth/middleware.ts tests/auth/jwt.test.ts
```

```bash
git commit -m "feat(auth): JWT ベースの認証を実装する

既存のセッションベース認証を置き換えるために JWT を導入した。

Co-Authored-By: <model-name> <noreply@anthropic.com>"
```

---

### コミット 2: `chore: ESLint 設定を更新する`

```bash
git add .eslintrc.json
```

```bash
git commit -m "chore: ESLint 設定を更新する

Co-Authored-By: <model-name> <noreply@anthropic.com>"
```
````

「--no-gpg-sign でコミットする」が選択された場合、グループ順にコミットを作成する。

**本文が必要な場合**（変更の背景、理由、詳細を伝える必要がある場合）:

```bash
git add <file1> <file2> ...
git commit [--no-gpg-sign] -m "$(cat <<'EOF'
<type>(<scope>): <subject>

<変更の背景・理由・詳細>

Co-Authored-By: <model-name> <noreply@anthropic.com>
EOF
)"
```

**本文が不要な場合**（件名だけで意図が伝わる場合）:

```bash
git add <file1> <file2> ...
git commit [--no-gpg-sign] -m "$(cat <<'EOF'
<type>(<scope>): <subject>

Co-Authored-By: <model-name> <noreply@anthropic.com>
EOF
)"
```

`[--no-gpg-sign]` は GPG フラグが立っており、ユーザーが承認した場合のみ含める。

すべてのグループが完了したら、`git log --oneline -N`（N = 作成したコミット数）でコミットを確認して一覧を報告する:

```
✓ 2 件のコミットを作成しました

  abc1234 feat(auth): JWT ベースの認証を実装する
  def5678 chore: ESLint 設定を更新する
```

---

## コミットメッセージの書き方

Conventional Commits 形式を使用する:

```
<type>(<scope>): <subject>

<body>
```

- **subject**: コミットの概要を 1 行で簡潔に記述する
- **body**: 変更の背景、理由、詳細が必要な場合のみ記述し、不要な場合は省略する

| type     | 使用場面                                             |
|----------|------------------------------------------------------|
| feat     | 新機能                                               |
| fix      | バグ修正                                             |
| refactor | リファクタリング（振る舞いの変更なし）               |
| test     | テストの追加、更新                                   |
| docs     | ドキュメントの変更                                   |
| chore    | ビルド、CI、設定、その他のハウスキーピング           |
| style    | フォーマットの変更（振る舞いの変更なし）             |

メッセージはプロジェクトで使用している言語で記述する（日本語プロジェクトなら日本語で記述する）。

---

## 注意事項

- **言語**: スキルを呼び出したときにユーザーが使用した言語で常に応答する
- **プッシュしない**: コミットのみを作成する。プッシュはユーザーの判断に委ねる。
- **バイナリファイル**: 差分を読めないため、ファイル名と用途から意図を推定し、ユーザーに確認する。
- **大規模な変更セット（50 ファイル以上）**: まず変更全体の概要を説明し、グループ化の方針を相談する。
- **センシティブなファイル**: `.env` などの認証情報を含むファイルをコミットに含める前に警告する。
- **pre-commit フックが失敗した場合**: フックのエラー出力を確認し、問題を修正してコミットを再試行する。ユーザーが明示的に要求した場合のみ `--no-verify` を提案する。
- **その他の理由でコミットが失敗した場合**: 生のエラー出力を表示し、再試行前に原因を診断する。よくあるケース: `index.lock` がすでに存在する（別の git プロセスが実行中。他のプロセスが保持していないことを確認してから `.git/index.lock` の削除を提案する）、ファイルのパーミッションエラー（影響するパスを報告する）、ディスク容量不足エラー（即座に報告して停止する）。
- **コミットするものがない**: ステージするファイルがないことをユーザーに伝え、`git status` の出力を表示する。
- **Co-Authored-By の正確な帰属**: コミットテンプレートの `<model-name>` を現在使用している正確なモデル名（例: `Claude Sonnet 4.6`）に置き換える。

