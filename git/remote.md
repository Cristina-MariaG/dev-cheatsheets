# Git — Remotes

> Working with remote repositories.

---

## Setup

```bash
git remote -v                           # list remotes
git remote add origin <url>             # add a remote
git remote set-url origin <url>         # change remote URL
git remote remove <name>                # remove a remote
```

---

## Fetch, Pull, Push

```bash
git fetch                               # download changes, don't merge
git fetch --prune                       # also remove deleted remote branches
git pull                                # fetch + merge
git pull --rebase                       # fetch + rebase (cleaner history)
git push origin <branch>               # push a branch
git push -u origin <branch>            # push and set upstream
git push --force-with-lease            # safe force push (checks remote state)
```

> **Prefer `--force-with-lease` over `--force`** — it won't overwrite if someone else pushed in the meantime.

---

## Tracking branches

```bash
git branch -u origin/<branch>          # set upstream for current branch
git branch -vv                         # show tracking info for all branches
```

---

## Common pitfalls

- `git push --force` on shared branches — overwrites teammates' work
- Pulling without `--rebase` creates unnecessary merge commits on long-lived branches
- Not pruning stale remote-tracking refs (`--prune`)
