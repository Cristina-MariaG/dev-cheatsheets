# Example — Feature Branch Workflow

> Standard workflow for a feature developed in isolation and merged via PR.

---

```bash
# 1. Start from an up-to-date main
git switch main
git pull --rebase

# 2. Create a feature branch
git switch -c feature/user-auth

# 3. Work, commit regularly
git add -p
git commit -m "feat: add JWT middleware"

# 4. Keep up with main (rebase, not merge)
git fetch origin
git rebase origin/main

# 5. Clean up history before PR
git rebase -i origin/main       # squash/fixup WIP commits

# 6. Push and open PR
git push -u origin feature/user-auth

# 7. After merge, clean up
git switch main
git pull --rebase
git branch -d feature/user-auth
```

---

**Why rebase instead of merge when updating?**
Keeps the branch history linear — easier to review and bisect later.
