# Resolving Merge Conflicts

## Problem / Context

When merging or rebasing branches, Git may be unable to automatically reconcile changes made to the same lines of a file (or when one branch deletes a file another branch modifies). This halts the merge/rebase process and requires manual intervention before work can continue.

Typical trigger scenarios:
- Two branches modify the same line(s) of a file differently.
- `git pull` fails because local commits diverge from the remote branch.
- A file is deleted in one branch but edited in another.

## Root Cause

Git's automatic three-way merge algorithm compares the common ancestor commit against both branch tips. When it cannot determine which version of a change should take precedence, it stops and inserts **conflict markers** directly into the affected file(s), leaving resolution to the developer.

## Resolution / Steps

### 1. Identify Conflicted Files

After a failed merge, Git reports which files are in conflict:

```bash
git status
```

Files listed under `Unmerged paths` require manual resolution.

### 2. Inspect the Conflict Markers

Open the conflicted file. Git inserts markers to delineate the competing versions:

```text
<<<<<<< HEAD
Your current branch's version of the code
=======
Incoming branch's version of the code
>>>>>>> feature/branch-name
```

- Content between `<<<<<<< HEAD` and `=======` is your current branch.
- Content between `=======` and `>>>>>>> branch-name` is the incoming branch.

### 3. Resolve the Conflict

Manually edit the file to keep the correct code, combine both changes, or discard one side entirely. **Remove all conflict markers** (`<<<<<<<`, `=======`, `>>>>>>>`) once resolved — leaving them in place will break the code.

### 4. Stage the Resolved File

Once the file reflects the intended final state:

```bash
git add path/to/resolved-file.js
```

Repeat for every conflicted file, then confirm nothing remains unresolved:

```bash
git status
```

### 5. Complete the Merge

If resolving a **merge**:

```bash
git commit
```

Git pre-fills a merge commit message — edit if needed, then save.

If resolving a **rebase**:

```bash
git rebase --continue
```

Repeat steps 2–5 for each conflicting commit in the rebase sequence.

### 6. Abort if Needed

To back out entirely and return to the pre-merge/pre-rebase state:

```bash
# During a merge
git merge --abort

# During a rebase
git rebase --abort
```

Use this when the conflict resolution becomes too complex or was started by mistake.

---

## Useful Diagnostic Commands

**View which commit last modified a conflicting line:**

```bash
git log --oneline -p -- path/to/file.js
```

**Use a visual merge tool (if configured):**

```bash
git mergetool
```

**See a side-by-side diff of both conflicting versions:**

```bash
git diff --ours path/to/file.js
git diff --theirs path/to/file.js
```

---

## Prevention Tips

- Pull frequently (`git pull`) to keep your branch in sync and minimize divergence.
- Keep feature branches short-lived and merge back regularly.
- Communicate with teammates when editing shared files in parallel.
- Use `git fetch` + `git diff origin/main` to preview incoming changes before merging.
