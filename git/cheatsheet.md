# Git — Cheatsheet

> Most used commands, grouped by situation.

---

## Start / resume work

```bash
git status                          # what's going on right now
git log --oneline --graph --all     # visual overview of branches
git stash list                      # any saved work?
git fetch origin                    # get remote changes (no merge)
git pull --rebase                   # update current branch
```

---

## Stage & commit

```bash
git add <file>                      # stage a file
git add -p                          # stage hunk by hunk (recommended)
git diff --staged                   # review what's about to be committed
git commit -m "type: message"       # commit
git commit --amend --no-edit        # add forgotten change to last commit
```

---

## Branches

```bash
git switch <branch>                 # switch branch
git switch -c <branch>             # create and switch
git branch -d <branch>             # delete merged branch
git branch -vv                     # show branches + tracking info
```

---

## Sync with remote

```bash
git push -u origin <branch>        # push and set upstream
git push --force-with-lease        # safe force push
git fetch --prune                  # clean up deleted remote branches
```

---

## Cherry-pick

```bash
git cherry-pick <hash>              # apply a specific commit to current branch
git cherry-pick <hash1>^..<hash2>  # apply a range of commits (inclusive)
git cherry-pick --no-commit <hash>  # apply changes without committing
```

---

## Stash

```bash
git stash push -m "message"        # save work in progress
git stash pop                      # apply last stash and remove it from the list
git stash list                     # see all stashes
```

---

## Fix a mistake

```bash
git restore <file>                 # discard unstaged changes
git restore --staged <file>        # unstage a file
git reset --soft HEAD~1            # undo last commit, keep changes staged
git revert <hash>                  # undo a pushed commit safely
git reflog                         # find anything lost
```

---

## Inspect

```bash
git log --oneline --graph --all    # full branch overview
git diff                           # unstaged changes
git diff --staged                  # staged changes
git diff <branch1>..<branch2>      # diff between branches
git log -S "keyword"               # find when a string was added/removed
git blame <file>                   # who changed what line
```
