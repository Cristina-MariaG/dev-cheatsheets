# Git — Advanced

> Hooks, worktrees, submodules, and debugging tools.

---

## Hooks

Hooks live in `.git/hooks/`. Make them executable: `chmod +x .git/hooks/<hook>`.

| Hook | Trigger | Use case |
|---|---|---|
| `pre-commit` | Before commit | Lint, format check |
| `commit-msg` | After message written | Enforce message format |
| `pre-push` | Before push | Run tests |
| `post-merge` | After merge/pull | Install dependencies |

```bash
#!/bin/sh
# .git/hooks/pre-commit — example: block commits with console.log
if git diff --cached | grep -q "console.log"; then
  echo "Error: console.log found. Remove before committing."
  exit 1
fi
```

> Use tools like **husky** or **lefthook** to share hooks across teams (`.git/hooks` is not committed).

---

## Worktrees

Check out multiple branches simultaneously without stashing.

```bash
git worktree add ../hotfix hotfix/urgent   # create a new worktree
git worktree list                          # list all worktrees
git worktree remove ../hotfix             # remove a worktree
```

> Useful for working on a hotfix while keeping your feature branch intact.

---

## Submodules

```bash
git submodule add <url> <path>            # add a submodule
git submodule update --init --recursive   # clone + init all submodules at their pinned commit
git submodule update --remote             # update all submodules to their latest remote commit
```

> **Pitfall:** Submodules pin to a specific commit. `update --remote` moves the pin forward — always commit the parent repo afterwards so others get the same version.

---

## Bisect — find the bad commit

```bash
git bisect start
git bisect bad                  # mark current commit as broken
git bisect good <hash>          # mark last known good commit
# Git checks out a midpoint — test it, then mark it:
git bisect good                 # midpoint is fine, bug is more recent
git bisect bad                  # midpoint is broken, bug is older
# repeat until Git identifies the culprit commit
git bisect reset                # exit bisect mode and return to original branch
```

---

## Useful one-liners

```bash
git shortlog -sn                          # contributors by commit count
git log --all --grep="keyword"            # search commit messages
git log -S "function_name"               # find when a string was added/removed
git diff --stat HEAD~5..HEAD             # summary of last 5 commits
git clean -fd                            # remove untracked files and dirs (destructive — no reflog recovery)
```
