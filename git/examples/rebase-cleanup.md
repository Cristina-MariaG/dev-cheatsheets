# Example — Rebase Cleanup Before PR

> Squash WIP commits and produce a clean, reviewable history.

---

```bash
# 1. See what commits will be included in the PR
git log --oneline origin/main..HEAD

# 2. Make sure origin/main is up to date before rebasing
git fetch origin

# 3. Interactive rebase
git rebase -i origin/main
```

In the editor, you'll see something like:

```
pick a1b2c3 feat: scaffold auth module
pick d4e5f6 wip: fix typo
pick 789abc wip: forgot to add file
pick def012 fix: handle edge case
pick 345678 wip: better variable name
```

Rewrite it as:

```
pick a1b2c3 feat: scaffold auth module
fixup d4e5f6 wip: fix typo
fixup 789abc wip: forgot to add file
pick def012 fix: handle edge case
reword 345678 wip: better variable name
```

- `fixup` absorbs the WIP commits silently into the previous commit
- `reword` opens the editor so you can write a proper message for the last one

```bash
# 4. Force push (safe — branch is yours and no one else is on it)
git push --force-with-lease origin feature/my-feature
```

---

## Rebase commands

| Command | Effect |
|---|---|
| `pick` | keep commit as-is |
| `reword` | keep commit, edit message |
| `squash` | merge into previous, combine both messages |
| `fixup` | merge into previous, discard message |
| `drop` | remove commit entirely |

---

## Pitfalls

- **Rebasing on stale main** — always `git fetch origin` before `git rebase -i origin/main`, otherwise you rebase on an outdated base.
- **Others checked out your branch** — force pushing rewrites hashes and breaks their local copies. Only rebase solo branches.
- **Rebase conflicts** — resolve each conflict, then `git rebase --continue`. Run `git rebase --abort` to cancel entirely and start over.
