# Git — Basics

> Essential commands for daily use.

---

## Init & Clone

```bash
git init                        # initialize a new repo
git clone <url>                 # clone a remote repo
git clone <url> --depth 1       # shallow clone (faster, no history)
```

---

## Staging & Committing

```bash
git status                      # show working tree status
git add <file>                  # stage a specific file
git add .                       # stage all changes
git add -p                      # stage interactively (hunk by hunk)
git commit -m "message"         # commit with message
git commit --amend              # edit last commit (message or content)
git commit --amend --no-edit    # amend without changing message
```

> **Pitfall:** `--amend` rewrites history — never use on already pushed commits.

---

## Log & Diff

```bash
git log                         # full commit history
git log --oneline --graph       # compact visual graph
git log --oneline --graph --all # same, including all branches
git log -p                      # show patch for each commit
git diff                        # unstaged changes
git diff --staged               # staged changes
git diff <branch1>..<branch2>   # diff between two branches
```

---

## Common pitfalls

- Forgetting `git add -p` and staging unintended changes
- Using `--amend` on pushed commits (forces others to rebase)
- Committing secrets — use `.gitignore` and `git secret` or similar tools
