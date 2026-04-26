---
name: composing-commits
description: >
  Analyzes changed files using git commands and creates commits at an appropriate
  level of granularity in logical units.
  Use when git changes (staged or unstaged) exist and need to be grouped and committed.
allowed-tools:
  - Bash(git add *)
  - Bash(git branch *)
  - Bash(git commit *)
  - Bash(git config *)
  - Bash(git diff *)
  - Bash(git log *)
  - Bash(git status *)
  - Bash(git switch *)
  - Glob
  - Grep
  - Read
  - AskUserQuestion
---

# Composing Git Commits

Analyze changed files, group related changes together, and create commits at an appropriate level of granularity.

## Quick start

1. Run `/composing-commits` or say "commit this" with staged or unstaged changes present
2. The skill checks git status, branch, and recent log to understand context
3. Each changed file is read to infer intent and change type
4. A proposed commit plan (groups + messages) is presented — confirm or adjust
5. Commits are executed group by group; a final log summary is shown

## Steps

### Step 1: Assess the current state

Check the changes and the project's commit style:

```bash
git branch --show-current
git config --get commit.gpgsign
git status
git diff --stat HEAD
git log --oneline -10
```

**If GPG signing is enabled (`commit.gpgsign=true`), set a flag.**
Do not prompt the user at this point. Confirm after Step 4 approval, before executing Step 5.

**Always confirm with the user when on the main or master branch.**
If the current branch is `main` or `master`, use AskUserQuestion to confirm before committing:

```javascript
AskUserQuestion({
  questions: [{
    question: "You are currently on the main branch. Do you want to commit directly to it?",
    header: "Branch Confirmation",
    options: [
      { label: "Commit to main",          description: "Commit directly to the current branch" },
      { label: "Create a working branch", description: "Create a new branch before committing" }
    ],
    multiSelect: false
  }]
})
```

If "Create a working branch" is selected, ask for a branch name as a follow-up text question, run `git switch -c <branch>` with the specified name, then continue.

If there are already staged files, confirm the user's intent before proceeding:
- Commit as-is (go to Step 4)
- Analyze including unstaged changes (go to Step 2)

### Step 2: Read the changes and understand the intent

For each file identified by the stat output in Step 1, read the diff to understand the intent behind the change. Analyze from the following angles:

- **Type of change**: feature addition / bug fix / refactoring / test addition / config change / documentation update
- **Purpose of change**: why it was changed (the axis for deciding whether changes belong together)
- **Relationship between files**: look for combinations that make sense when changed together

```bash
git diff HEAD -- <file>
```

Untracked new files are not included in `git diff HEAD`, so use Read to inspect their contents.

### Step 3: Group changes and draft a commit plan

Group changes according to the following principles:

**Changes that can be grouped together**
- Source files and tests belonging to the same feature or purpose
- A series of files for the same bug fix
- Files targeted by the same refactoring

**Changes that should be separate commits**
- Different purposes (feature addition vs. bug fix vs. refactoring, etc.)
- Independent of each other (neither depends on the other)
- Different areas of impact (application code / config / documentation / CI, etc.)

If a single changed file contains multiple intents mixed together, consider using `git add -p` for partial staging.

### Step 4: Present the plan to the user and get confirmation

**Always confirm before executing.** Propose in the following format:

```
Proposed commits (N total):

Commit 1: feat(auth): implement JWT-based authentication
  Files:
    - src/auth/jwt.ts (new)
    - src/auth/middleware.ts (modified)
    - tests/auth/jwt.test.ts (new)

Commit 2: chore: update ESLint configuration
  Files:
    - .eslintrc.json (modified)

Proceed with this plan?
```

If changes are requested, adjust the grouping and re-propose.

### Step 5: Execute commits for each group

Once approved, if the GPG flag is set, confirm with AskUserQuestion before committing:

```javascript
AskUserQuestion({
  questions: [{
    question: "GPG signing is enabled. Since Claude Code cannot enter a passphrase, commits in this session will use --no-gpg-sign. Is that okay?",
    header: "GPG Signing",
    options: [
      { label: "Commit with --no-gpg-sign", description: "Skip GPG signing for this session only" },
      { label: "Cancel",                    description: "Abort the process (please commit manually from your terminal)" }
    ],
    multiSelect: false
  }]
})
```

If "Cancel" is selected, output the commit plan confirmed in Step 4 as runnable commands and stop:

```
Aborted because GPG signing is required.
Please run the following commands from your terminal:

# Commit 1
git add src/auth/jwt.ts src/auth/middleware.ts tests/auth/jwt.test.ts
git commit -m "feat(auth): implement JWT-based authentication

Introduced JWT to replace the existing session-based authentication."

# Commit 2
git add .eslintrc.json
git commit -m "chore: update ESLint configuration"
```

If "Commit with --no-gpg-sign" is selected, create commits in group order.

**When a body is needed** (to provide background, rationale, or detail about the change):

```bash
git add <file1> <file2> ...
git commit [--no-gpg-sign] -m "$(cat <<'EOF'
<type>(<scope>): <subject>

<background, rationale, and details of the change>

Co-Authored-By: <model-name> <noreply@anthropic.com>
EOF
)"
```

**When a body is not needed** (when the subject alone conveys the intent):

```bash
git add <file1> <file2> ...
git commit [--no-gpg-sign] -m "$(cat <<'EOF'
<type>(<scope>): <subject>

Co-Authored-By: <model-name> <noreply@anthropic.com>
EOF
)"
```

`[--no-gpg-sign]` is only included when the GPG flag is set and the user has approved it.

After all groups are complete, verify the commits with `git log --oneline -N` (where N is the number of commits) and report the list:

```bash
git log --oneline -3
```

```
✓ 2 commits created

  abc1234 feat(auth): implement JWT-based authentication
  def5678 chore: update ESLint configuration
```

---

## How to write commit messages

Use the Conventional Commits format:

```
<type>(<scope>): <subject>

<body>
```

- **subject**: A concise one-line summary of the commit
- **body**: Include only when background, rationale, or detail about the change is needed; omit otherwise

| type     | Use case                                          |
|----------|---------------------------------------------------|
| feat     | New feature                                       |
| fix      | Bug fix                                           |
| refactor | Refactoring (no behavior change)                  |
| test     | Adding or updating tests                          |
| docs     | Documentation changes                             |
| chore    | Build, CI, configuration, and other housekeeping  |
| style    | Formatting changes (no behavior change)           |

Write messages in the language used by the project (e.g., use Japanese for a Japanese-language project).

---

## Notes

- **Language**: Always respond in the same language the user used to invoke the skill
- **Do not push**: Only create commits. Pushing is left to the user's discretion.
- **Binary files**: Since the diff cannot be read, infer the intent from the file name and purpose, then confirm with the user.
- **Large changesets (50+ files)**: First explain the overall picture of the changes, then discuss a grouping strategy.
- **Sensitive files**: Warn before including files with credentials such as `.env` in a commit.
- **If a pre-commit hook fails**: Review the hook's error output, fix the issue, and retry the commit. Only suggest `--no-verify` if the user explicitly requests it.
- **Nothing to commit**: Inform the user that there are no files to stage and show the output of `git status`.
- **Accurate Co-Authored-By attribution**: Replace `<model-name>` in the commit template with the exact model currently in use (e.g., `Claude Sonnet 4.6`).
