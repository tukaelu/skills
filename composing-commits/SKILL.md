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
  - Bash(git stash *)
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
3. Changes are analyzed to infer intent and change type
4. A proposed commit plan (groups + messages) is presented — confirm or adjust
5. Commits are executed group by group; a final log summary is shown

## Steps

### Step 1: Assess the current state

Check the changes and the project's commit style. These commands are independent — run them all in parallel:

```bash
git branch --show-current
git config --get commit.gpgsign
git status
git diff --stat HEAD
git log --oneline -10
git stash list
```

**If GPG signing is enabled (`commit.gpgsign=true`), set a flag.**
Do not prompt the user at this point. Confirm after Step 4 approval, before executing Step 5.

**If `git stash list` returns any entries**, inform the user before proceeding:

```
Note: you have N stashed change(s). Make sure they are intentionally excluded from this commit.
```

Do not block on this — just surface the information and continue.

**Always confirm with the user when on the main or master branch.**
If the current branch is `main` or `master`, use AskUserQuestion to confirm before committing:

```javascript
AskUserQuestion({
  questions: [{
    question: "You are currently on the main branch. How would you like to proceed?",
    header: "Branch Confirmation",
    options: [
      { label: "Create a working branch", description: "Create a new branch before committing" },
      { label: "Commit to main",          description: "Commit directly to the current branch" }
    ],
    multiSelect: false
  }]
})
```

If "Create a working branch" is selected, ask for a branch name as a follow-up text question, run `git switch -c <branch>` with the specified name, then continue.

If there are already staged files, confirm the user's intent with AskUserQuestion before proceeding:

```javascript
AskUserQuestion({
  questions: [{
    question: "Some files are already staged. How would you like to proceed?",
    header: "Staged Files",
    options: [
      { label: "Analyze all changes",       description: "Include unstaged changes in the analysis (Step 2)" },
      { label: "Commit as-is",              description: "Commit only the already-staged files (skips Step 2)" }
    ],
    multiSelect: false
  }]
})
```

If "Analyze all changes" is selected, continue to Step 2 as normal.

If "Commit as-is" is selected, run `git diff --cached` to understand the staged content. Treat all staged files as a single commit group, then skip to Step 4.

### Step 2: Read the changes and understand the intent

Retrieve all diffs in a single command rather than per-file:

```bash
git diff HEAD
```

Then analyze the output to understand the intent behind each change. Analyze from the following angles:

- **Type of change**: feature addition / bug fix / refactoring / test addition / config change / documentation update
- **Purpose of change**: why it was changed (the axis for deciding whether changes belong together)
- **Relationship between files**: look for combinations that make sense when changed together

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

**Always confirm before executing.** First output the proposed plan as text:

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
```

Then immediately follow with AskUserQuestion:

```javascript
AskUserQuestion({
  questions: [{
    question: "Proceed with this commit plan?",
    header: "Commit Plan",
    options: [
      { label: "Proceed", description: "Execute commits as proposed" },
      { label: "Modify",  description: "Adjust grouping or commit messages" },
      { label: "Cancel",  description: "Abort without committing" }
    ],
    multiSelect: false
  }]
})
```

If "Modify" is selected, ask a follow-up text question for specific changes, adjust the plan, and re-propose from the top of Step 4.

If "Cancel" is selected, stop immediately.

### Step 5: Execute commits for each group

Once approved, if GPG is not set, create commits in group order immediately.

If the GPG flag is set, confirm with AskUserQuestion before committing:

```javascript
AskUserQuestion({
  questions: [{
    question: "GPG signing is enabled. Since Claude Code cannot enter a passphrase, commits in this session will use --no-gpg-sign. Is that okay?",
    header: "GPG Signing",
    options: [
      { label: "Commit with --no-gpg-sign", description: "Skip GPG signing for this session only" },
      { label: "Run myself",                description: "Output commands to run manually in your terminal" },
      { label: "Cancel",                    description: "Abort without committing" }
    ],
    multiSelect: false
  }]
})
```

If "Cancel" is selected, stop without outputting anything further.

If "Run myself" is selected, output the commit plan as copy-paste-ready commands and stop. To avoid heredoc interpretation issues, **separate the `git add` and `git commit` commands into distinct blocks**. Use a plain quoted string for the message (no heredoc):

````
GPG signing is required. Please run the following commands from your terminal:

---

### Commit 1: `feat(auth): implement JWT-based authentication`

```bash
git add src/auth/jwt.ts src/auth/middleware.ts tests/auth/jwt.test.ts
```

```bash
git commit -m "feat(auth): implement JWT-based authentication

Introduced JWT to replace the existing session-based authentication.

Co-Authored-By: <model-name> <noreply@anthropic.com>"
```

---

### Commit 2: `chore: update ESLint configuration`

```bash
git add .eslintrc.json
```

```bash
git commit -m "chore: update ESLint configuration

Co-Authored-By: <model-name> <noreply@anthropic.com>"
```
````

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

After all groups are complete, verify the commits with `git log --oneline -N` (N = number of commits just created) and report the list:

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
- **If a commit fails for other reasons**: Show the raw error output and diagnose the cause before retrying. Common cases: `index.lock` already exists (another git process is running — suggest removing `.git/index.lock` after confirming no other process holds it), file permission errors (report the affected path), or disk-full errors (report immediately and stop).
- **Nothing to commit**: Inform the user that there are no files to stage and show the output of `git status`.
- **Accurate Co-Authored-By attribution**: Replace `<model-name>` in the commit template with the exact model currently in use (e.g., `Claude Sonnet 4.6`).
