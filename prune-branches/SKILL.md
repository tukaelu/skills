---
name: prune-branches
description: >
  ベースブランチにマージ済みのローカル作業ブランチを安全に削除する。
  リモートブランチの削除は明示的に指示された場合のみ実行する。
  マージ済みブランチの整理、PR マージ後の古いブランチ削除、ローカルやリモートの refs の整頓時に使用する。
allowed-tools:
  - Bash(git branch *)
  - Bash(git log *)
  - Bash(git remote *)
  - Bash(git push *)
  - Bash(git fetch *)
  - AskUserQuestion
---

# マージ済みブランチの削除

ベースブランチにマージ済みのローカル作業ブランチを安全に削除する。リモートブランチの削除は明示的に指示された場合のみ実行する。

## クイックスタート

1. `/prune-branches` を実行するか「マージ済みブランチを削除して」と伝える
2. ベースブランチを確認する（デフォルト: `main`）
3. マージ済みローカルブランチの一覧を確認 → 承認後に削除する
4. リモートブランチの削除は明示的に要求された場合のみ実行する

## ステップ

### ステップ 1: ベースブランチを特定する

```bash
git branch --show-current
git remote
```

ベースブランチの優先順位:
1. 引数またはメッセージでユーザーが指定したブランチ
2. `main` が存在する場合はそれを使用
3. `master` が存在する場合はそれを使用
4. どちらも存在しない場合は AskUserQuestion で尋ねる

```bash
git branch --list main master
```

### ステップ 2: マージ済みのローカルブランチを一覧表示する

```bash
git fetch --prune
git branch --merged <base-branch>
```

**常に除外するもの:**
- 現在チェックアウト中のブランチ（`*` が付いているもの）
- ベースブランチ自体（`main` / `master` など）
- 保護されたブランチ: `develop`, `staging`, `release`, `production`

除外後に候補が残らない場合は「削除するブランチがありません」と報告して停止する。

### ステップ 3: 候補を提示して確認を取る

候補を表示して AskUserQuestion で確認する:

```
削除するローカルブランチ（N 件）:

  - feature/add-login
  - fix/typo-in-readme
  - chore/update-deps

これらのブランチを削除しますか？
```

```javascript
AskUserQuestion({
  questions: [{
    question: "上記のブランチを削除しますか？",
    header: "ローカルブランチ削除の確認",
    options: [
      { label: "すべて削除する", description: "一覧のすべてのブランチを削除する" },
      { label: "選択して削除する", description: "ブランチを個別に選択する" },
      { label: "キャンセル",      description: "何も削除せずに終了する" }
    ],
    multiSelect: false
  }]
})
```

「選択して削除する」が選択された場合、ブランチ名を選択肢とする multiSelect の質問で続ける。

### ステップ 4: ローカルブランチを削除する

承認された各ブランチに対して安全な削除を実行する:

```bash
git branch -d <branch>
```

> `-D`（強制削除）は絶対に使用しない。`-d` はブランチが本当にマージ済みであることを確認する最後の安全策である。

`-d` が失敗した場合（ブランチが完全にマージされていないと報告された場合）:
- ユーザーに状況を報告する
- 強制削除を提案しない
- `git log <base>..<branch>` で確認するよう案内する

すべての削除が完了したら結果を報告する:

```
✓ 2 件のローカルブランチを削除しました

  削除済み: feature/add-login
  削除済み: fix/typo-in-readme
  スキップ: chore/update-deps（完全にマージされていないと報告されました）
```

### ステップ 5: リモートブランチの削除（明示的に指示された場合のみ）

**このステップはユーザーがリモートブランチの削除を明示的に要求した場合のみ実行する**（例: 「リモートも削除して」「リモートも整理して」）。自発的に提案しない。

削除前にマージ済みのリモートブランチを確認する:

```bash
git branch -r --merged <remote>/<base-branch>
```

**常に除外するもの:**
- `<remote>/HEAD`
- `<remote>/main`, `<remote>/master`, `<remote>/develop`, `<remote>/staging`, `<remote>/release`, `<remote>/production`

ステップ 3 と同様に候補を提示し、確認後に実行する:

```bash
git push <remote> --delete <branch>
```

すべての削除が完了したら、古いリモートトラッキング refs を整理する:

```bash
git fetch --prune
```

---

## 注意事項

- **`-D` は絶対に使用しない**: `git branch -d` のみを使用する。強制削除はユーザーが手動で判断する事項である。
- **リモートの削除はオプトイン**: 明示的な指示なしにリモートブランチの削除を提案したり、実行したりしない。
- **最初に `git fetch --prune` を実行する**: リモートですでに削除された refs をブランチ一覧から除外するために必要。
- **言語**: スキルを呼び出したときにユーザーが使用した言語で常に応答する。

