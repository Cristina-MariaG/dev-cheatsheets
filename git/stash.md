# Git — Stash

> Temporarily shelve changes without committing.

---

## Basic usage

```bash
git stash                       # stash tracked changes (untracked files NOT included by default)
git stash push -m "message"     # stash with a label
git stash pop                   # apply last stash and remove it
git stash apply                 # apply last stash, keep it in list
git stash list                  # show all stashes
git stash drop stash@{0}        # delete a specific stash
git stash clear                 # delete all stashes
```

---

## Partial stash

```bash
git stash push -p               # interactively choose hunks to stash
git stash push <file>           # stash a specific file only
```

---

## Stash with untracked files

```bash
git stash -u                    # include untracked files
git stash -a                    # include untracked + ignored files
```

---

## Apply to another branch

```bash
git switch <other-branch>
git stash pop                   # apply stash on the new branch
```

---

## Common pitfalls

- Losing stashes after `git stash clear` — refs are deleted, but objects may still be reachable via `git fsck --unreachable` before the next GC
- Stash conflicts when applying to a branch with diverging changes
- Forgetting `-u` and leaving untracked files behind
