# Example — Daily Scenarios

> Real situations encountered every day.

---

## Start the day

```bash
git status                         # check current state
git fetch origin                   # get remote changes
git pull --rebase                  # update current branch
```

---

## I committed on the wrong branch

```bash
git log --oneline -1               # note the commit hash
git switch correct-branch
git cherry-pick <hash>

git switch wrong-branch
git reset --hard HEAD~1            # remove it (only if not pushed)
```

---

## Main moved forward, I need to update my branch

```bash
git fetch origin
git rebase origin/main             # replay my commits on top of new main
# if conflicts:
#   resolve, then git rebase --continue
#   or git rebase --abort to cancel
```

---

## I need to switch branches but have unfinished work

```bash
git stash push -m "wip: what I was doing"
git switch other-branch
# do the work...
git switch -
git stash pop
```

---

## I want to undo my last commit but keep the changes

```bash
git reset --soft HEAD~1            # commit is gone, changes are staged
```

---

## There's a conflict after a merge or rebase

```bash
git status                         # see which files conflict
# open the files, resolve the markers:
# <<<<<<< HEAD
# your version
# =======
# incoming version
# >>>>>>> branch-name

git add <resolved-file>
git rebase --continue              # if the conflict came from a rebase
# git merge --continue             # if the conflict came from a merge
```

---

## I want to check what changed before committing

```bash
git diff                           # unstaged changes
git diff --staged                  # staged changes
git log --oneline origin/main..HEAD  # commits not yet on main
```

---

## End of day — save work in progress

```bash
# Option 1 — stash (no commit)
git stash push -m "wip: end of day"

# Option 2 — WIP commit (easy to undo tomorrow)
git add -p
git commit -m "wip: end of day"
# tomorrow: git reset --soft HEAD~1 to undo before continuing
```
