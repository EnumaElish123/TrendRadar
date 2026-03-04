---
name: git-push-helper
description: >
  Pushes local commits to a remote repository following safe push practices.
  Use when user asks to push code, sync with remote, or mentions "git push",
  "push to GitHub", or "upload changes". Handles upstream setup, conflict
  detection, and common push failure scenarios.
---

# Git Push Helper

## Workflow

1. Run `git status` to confirm the working tree is clean and commits exist
2. Run `git remote -v` to identify the remote name and URL
3. Run `git branch -vv` to check the current branch and upstream tracking status
4. Determine the correct push command from the **Decision Guide** below
5. Execute the push command
6. Confirm success or diagnose the failure

---

## Decision Guide

### Step 1 — Does the branch have an upstream set?

```bash
git branch -vv
```

- If output shows `[origin/branchname]` → upstream is set, go to Step 2
- If output shows no `[...]` → upstream is NOT set, use:

```bash
# Long form
git push --set-upstream origin <branch-name>
# Shorthand (equivalent)
git push -u origin <branch-name>
```

### Step 2 — Fetch and check divergence

Always fetch before pushing so you can see the true remote state:

```bash
git fetch origin
git status
```

Read the output:

- `Your branch is ahead of 'origin/...' by N commits` → local has new commits, remote has nothing new → safe to push, go to Step 3
- `Your branch is behind 'origin/...'` → remote has commits local hasn't pulled yet → do NOT push yet; see **Conflict Scenarios**
- `Your branch and 'origin/...' have diverged` → both sides have unique commits → see **Conflict Scenarios**
- `Your branch is up to date` → nothing to push

### Step 3 — Execute push

Standard push (most common case):

```bash
git push origin <branch-name>
```

---

## Conflict Scenarios

### Remote is ahead (non-diverged)

Local and remote have not diverged; remote simply has newer commits:

```bash
git pull --rebase origin <branch-name>
# Resolve any rebase conflicts if prompted, then:
git push origin <branch-name>
```

### Branches have diverged

Both local and remote have unique commits:

```bash
git fetch origin
git log --oneline HEAD..origin/<branch-name>  # see what remote has
git log --oneline origin/<branch-name>..HEAD  # see what local has
# Then decide: rebase (cleaner history) or merge
git pull --rebase origin <branch-name>        # recommended
git push origin <branch-name>
```

### Push rejected (non-fast-forward)

```
! [rejected]  master -> master (non-fast-forward)
```

> ⚠️ **Never use `--force` on shared branches.** Use `--force-with-lease`
> only on your own private feature branches, after confirming no teammates
> have pushed to it.

```bash
# Safe option: rebase then push
git pull --rebase origin <branch-name>
git push origin <branch-name>

# Last resort on a private branch only:
git push --force-with-lease origin <branch-name>
```

---

## Multiple Remotes

If `git remote -v` shows more than one remote (e.g., `origin` and `upstream`), be explicit:

```bash
git remote -v          # list all remotes and their URLs
git push origin <branch-name>      # push to your fork
git push upstream <branch-name>    # push to the canonical repo
```

Never assume a single remote exists. Always check first.

---

## Pushing Tags

Tags are **not** pushed by default. Use one of:

```bash
# Push a single tag
git push origin v1.2.0

# Push all local tags at once
git push origin --tags

# Annotated tags only (recommended for releases)
git push origin --follow-tags
```

> Prefer `--follow-tags` for releases — it pushes only annotated tags reachable from pushed commits, avoiding accidental tag floods.

---

## Common Errors & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `fatal: 'origin' does not appear to be a git repository` | Remote URL wrong or not set | `git remote add origin <url>` |
| `error: src refspec main does not match any` | Branch is named `master`, not `main` | `git push origin master` |
| `remote: Repository not found` | No access or wrong URL | Check URL and GitHub auth |
| `! [rejected] ... (fetch first)` | Remote has new commits | `git pull --rebase`, then push |
| `Permission denied (publickey)` | SSH key not configured | Switch to HTTPS or add SSH key |

---

## Examples

```bash
# Standard push on a tracked branch
git push origin master

# First push of a new branch (long form)
git push --set-upstream origin feature/my-feature
# Shorthand equivalent
git push -u origin feature/my-feature

# After resolving a diverged remote
git pull --rebase origin master
git push origin master

# Push a release tag
git push origin v1.0.0

# Push all annotated tags reachable from current commits
git push origin --follow-tags
```
