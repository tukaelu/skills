---
name: pruning-branches
description: >
  Safely deletes local work branches that have been merged into a base branch.
  Optionally deletes remote branches only when explicitly instructed.
  Use when cleaning up merged branches, removing stale branches after a merge or PR, or tidying up local/remote refs.
allowed-tools:
  - Bash(git branch *)
  - Bash(git log *)
  - Bash(git remote *)
  - Bash(git push *)
  - Bash(git fetch *)
  - AskUserQuestion
---

# Pruning Merged Branches

Safely delete local work branches that have been merged into a base branch. Remote branches are deleted only when explicitly instructed.

## Quick start

1. Run `/pruning-branches` or say "delete merged branches"
2. Confirm the base branch (default: `main`)
3. Review the list of merged local branches → delete after confirmation
4. Remote branch deletion is performed only when explicitly requested

## Steps

### Step 1: Identify the base branch

```bash
git branch --show-current
git remote
```

Base branch priority:
1. Branch specified by the user in the arguments or message
2. `main` if it exists
3. `master` if it exists
4. If neither exists, ask via AskUserQuestion

```bash
git branch --list main master
```

### Step 2: List merged local branches

```bash
git fetch --prune
git branch --merged <base-branch>
```

**Always exclude:**
- The currently checked-out branch (marked with `*`)
- The base branch itself (`main` / `master`, etc.)
- Protected branches: `develop`, `staging`, `release`, `production`

If no candidates remain after exclusions, report "No branches to delete" and stop.

### Step 3: Present candidates and get confirmation

Display the candidates and ask via AskUserQuestion:

```
Local branches to delete (N):

  - feature/add-login
  - fix/typo-in-readme
  - chore/update-deps

Delete these branches?
```

```javascript
AskUserQuestion({
  questions: [{
    question: "Delete the branches listed above?",
    header: "Confirm local branch deletion",
    options: [
      { label: "Delete all",    description: "Delete every branch in the list" },
      { label: "Choose",        description: "Select branches individually" },
      { label: "Cancel",        description: "Exit without deleting anything" }
    ],
    multiSelect: false
  }]
})
```

If "Choose" is selected, follow up with a multiSelect question listing the branch names.

### Step 4: Delete local branches

Run the safe delete for each approved branch:

```bash
git branch -d <branch>
```

> Never use `-D` (force delete). `-d` is the last safeguard that confirms a branch is truly merged.

If `-d` fails (branch reported as not fully merged):
- Report the situation to the user
- Do not suggest force deletion
- Guide them to inspect with `git log <base>..<branch>`

Report the outcome after all deletions:

```
✓ 2 local branch(es) deleted

  Deleted: feature/add-login
  Deleted: fix/typo-in-readme
  Skipped: chore/update-deps (reported as not fully merged)
```

### Step 5: Delete remote branches (only when explicitly instructed)

**Execute this step only when the user explicitly asks to delete remote branches** (e.g. "also delete remote", "clean up the remote too"). Do not suggest it unprompted.

Check which remote branches are merged before deleting:

```bash
git branch -r --merged <remote>/<base-branch>
```

**Always exclude:**
- `<remote>/HEAD`
- `<remote>/main`, `<remote>/master`, `<remote>/develop`, `<remote>/staging`, `<remote>/release`, `<remote>/production`

Present the candidates the same way as Step 3, then execute after confirmation:

```bash
git push <remote> --delete <branch>
```

After all deletions, clean up stale remote-tracking refs:

```bash
git fetch --prune
```

---

## Notes

- **Never use `-D`**: Only `git branch -d` is allowed. Force deletion is a decision for the user to make manually.
- **Remote deletion is opt-in**: Do not propose or perform remote branch deletion without explicit instruction.
- **Run `git fetch --prune` first**: Ensures the branch list excludes refs already deleted on the remote.
- **Language**: Always respond in the same language the user used to invoke the skill.
