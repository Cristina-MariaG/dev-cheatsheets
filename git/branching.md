# Git — Branching

> Managing branches, merging strategies, and rebase workflows.

---

## Branches

> `git switch` and `git restore` were introduced in Git 2.23 (2019) to replace `git checkout`, which did too many things at once. `switch` handles branch changes, `restore` handles file changes. `checkout` still works but prefer the modern commands.
>
> | Old | Modern |
> |---|---|
> | `git checkout <branch>` | `git switch <branch>` |
> | `git checkout -b <branch>` | `git switch -c <branch>` |
> | `git checkout -- <file>` | `git restore <file>` |

```bash
git branch                      # list local branches
git branch -a                   # list local + remote branches
git branch <name>               # create a branch
git switch <name>               # switch to a branch
git switch -c <name>            # create and switch
git branch -d <name>            # delete merged branch
git branch -D <name>            # force delete
```

---

## Merge

```bash
git merge <branch>              # merge into current branch
git merge --no-ff <branch>      # force a merge commit (preserves history)
git merge --squash <branch>     # squash all commits into one staged change
```

| Strategy | When to use |
|---|---|
| `--ff` (default) | Clean linear history, small branches |
| `--no-ff` | Preserve branch context (feature branches) |
| `--squash` | Clean up messy WIP commits before merging |

---

## Rebase

```bash
git rebase <branch>             # rebase current branch onto another
git rebase -i HEAD~3            # interactive rebase on last 3 commits
git rebase --abort              # cancel an ongoing rebase
git rebase --continue           # continue after resolving conflicts
```

### Interactive rebase commands

| Command | Effect |
|---|---|
| `pick` | keep commit as-is |
| `reword` | keep commit, edit message |
| `squash` | merge into previous commit |
| `fixup` | like squash, discard message |
| `drop` | remove commit entirely |

> **Pitfall:** Never rebase shared/public branches — it rewrites commit hashes and breaks everyone else's history.

---

## Cherry-pick

```bash
git cherry-pick <hash>          # apply a specific commit to current branch
git cherry-pick <hash1>^..<hash2> # apply a range of commits (inclusive)
git cherry-pick --no-commit <hash> # apply changes without committing
```

---

## Common pitfalls

- Rebasing public branches (rewrite history = chaos for teammates)
- Forgetting to delete stale branches after merge
- Merge conflicts ignored with `--theirs` / `--ours` without understanding the impact
