# Git — Undoing Changes

> How to fix mistakes at every stage.

---

## Overview

| Situation | Command |
|---|---|
| Discard unstaged changes | `git restore <file>` |
| Unstage a file | `git restore --staged <file>` |
| Undo last commit, keep changes staged | `git reset --soft HEAD~1` |
| Undo last commit, keep changes unstaged | `git reset --mixed HEAD~1` |
| Undo last commit, discard changes | `git reset --hard HEAD~1` |
| Undo a pushed commit safely | `git revert <hash>` |

---

## Reset

```bash
git reset --soft HEAD~1         # undo commit, keep changes staged
git reset --mixed HEAD~1        # undo commit, keep changes unstaged (default)
git reset --hard HEAD~1         # undo commit, discard all changes
```

> **Pitfall:** `--hard` is destructive and unrecoverable without `reflog`.

---

## Revert

```bash
git revert <hash>               # create a new commit that undoes a previous one
git revert HEAD                 # revert last commit
git revert --no-commit <hash>   # stage the revert without committing
```

> Use `revert` (not `reset`) on pushed commits — it preserves history.

---

## Reflog — the safety net

```bash
git reflog                      # show all recent HEAD positions
git reset --hard HEAD@{2}       # jump back to a specific reflog entry
```

> `reflog` saves you from `--hard` resets and bad rebases. It retains entries for ~90 days by default.

---

## Restore

```bash
git restore <file>              # discard unstaged changes
git restore --staged <file>     # unstage without losing changes
git restore --source=<hash> <file>  # restore a file from a specific commit
```
