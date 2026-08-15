# Git General Workflow Cheat Sheet

## Problem / Context

Daily development requires a consistent set of Git commands to initialize repositories, track changes, synchronize with remotes, and manage tags. This guide consolidates the most frequently used commands into a single quick-reference cheat sheet to reduce context-switching and command lookup time.

## Root Cause

Not applicable — this is a reference guide rather than a troubleshooting document. It exists to standardize daily Git usage across the team and minimize inconsistent workflows (e.g., forgetting to stage files, pushing to the wrong branch, or leaving stale tags on the remote).

## Resolution / Steps

### 1. Repository Initialization

**`git init`**
Initializes a new local Git repository in the current directory. Creates a hidden `.git/` folder that tracks version history.

```bash
git init
```

Use when starting a brand-new project that isn't yet under version control.

---

### 2. Checking Status

**`git status`**
Shows the current state of the working directory: staged changes, unstaged changes, and untracked files.

```bash
git status
```

Use this **before every commit** to verify exactly what will be included.

---

### 3. Staging Changes

**`git add .`**
Stages all modified and new files in the current directory (and subdirectories) for the next commit.

```bash
git add .
```

For more granular control, stage specific files instead:

```bash
git add path/to/file.js
```

---

### 4. Committing Changes

**`git commit -m`**
Records the staged changes as a new commit with a descriptive message.

```bash
git commit -m "Add user authentication middleware"
```

**Best practice:** Write commit messages in the imperative mood (e.g., "Fix", "Add", "Refactor") and keep the first line under ~50 characters.

---

### 5. Remote Operations & Synchronization

**`git pull`**
Fetches changes from the remote repository and merges them into the current local branch. Use this at the start of a work session to sync with the latest changes from teammates.

```bash
git pull origin main
```

**`git push`**
Uploads local commits to the remote repository, making them available to others.

```bash
git push origin main
```

If pushing a new local branch for the first time, set the upstream tracking branch:

```bash
git push -u origin feature/new-login-flow
```

---

### 6. Tag Management

Tags mark specific points in history (typically releases, e.g., `v1.0.0`).

**Creating a tag**

```bash
git tag v1.0.0
```

For an annotated tag (recommended for releases — includes metadata like author and date):

```bash
git tag -a v1.0.0 -m "Release version 1.0.0"
```

Tags must be pushed explicitly; they are **not** included in a regular `git push`:

```bash
git push origin v1.0.0
```

To push all local tags at once:

```bash
git push origin --tags
```

**Deleting a remote tag**

```bash
git push origin --delete v1.0.0
```

Use when a tag was pushed by mistake or a release needs to be re-cut.

**Deleting a local tag**

```bash
git tag -d v1.0.0
```

**Note:** Deleting a local tag does **not** remove it from the remote — both commands are typically required together when correcting a mistaken release tag.

**Tag List**
```bash
git tag -l
```
or
```bash
git tag --list
```
Display All Tags: If run without additional arguments, this command will list all existing tags in sequence.

```bash
# Only show tags for version 1.x
git tag -l "v1.*"
```
or
```bash
# Only show tags containing the word "beta"
git tag -l "*beta*"
```
Filtering Tags Using Specific Patterns (Wildcards): You can search for tags that match a specific pattern using an asterisk (*).\
Note: When using a wildcard pattern, the -l or --list argument is mandatory.

---

### 7. Essential Daily-Use Utilities

**Branching**

Create and switch to a new branch:

```bash
git checkout -b feature/payment-integration
```

Switch to an existing branch:

```bash
git checkout main
```

List all local branches:

```bash
git branch
```

Delete a local branch (after it's merged):

```bash
git branch -d feature/payment-integration
```

**Viewing history**

Show a compact, one-line-per-commit log:

```bash
git log --oneline --graph --decorate
```

Show the last 5 commits with full details:

```bash
git log -5
```

**Inspecting changes**

Show unstaged differences between the working directory and the last commit:

```bash
git diff
```

Show staged differences (what will be committed):

```bash
git diff --staged
```

**Undoing changes**

Unstage a file without discarding changes:

```bash
git restore --staged path/to/file.js
```

Discard local (uncommitted) changes to a file:

```bash
git restore path/to/file.js
```

**Stashing work-in-progress**

Temporarily shelve uncommitted changes without committing:

```bash
git stash
```

Reapply the most recent stash:

```bash
git stash pop
```

---

## Quick Reference Table

| Command | Purpose |
|---|---|
| `git init` | Initialize a new repository |
| `git status` | Check working directory state |
| `git add .` | Stage all changes |
| `git commit -m "msg"` | Commit staged changes |
| `git pull origin main` | Sync local branch with remote |
| `git push origin main` | Upload commits to remote |
| `git tag v1.0.0` | Create a local tag |
| `git push origin v1.0.0` | Push a tag to remote |
| `git push origin --delete v1.0.0` | Delete a remote tag |
| `git tag -d v1.0.0` | Delete a local tag |
| `git checkout -b <branch>` | Create and switch to a new branch |
| `git log --oneline --graph` | View compact commit history |
| `git stash` / `git stash pop` | Shelve / restore uncommitted work |
