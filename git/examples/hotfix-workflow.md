# Example — Hotfix Workflow

> Urgent fix on production without disrupting ongoing feature work.

---

```bash
# 1. Save current work
git stash push -m "wip: current feature"

# 2. Start from main
git switch main
git pull --rebase

# 3. Create a hotfix branch
git switch -c hotfix/payment-crash

# 4. Fix, commit
git add -p
git commit -m "fix: prevent null pointer on empty cart"

# 5. Merge into main
git switch main
git merge --no-ff hotfix/payment-crash
git push origin main

# 6. Merge into your feature branch too (avoid reintroducing the bug)
git switch feature/my-feature
git rebase main

# 7. Clean up
git branch -d hotfix/payment-crash

# 8. Restore your work
git stash pop
```

---

**Why `--no-ff` here?**
Keeps the hotfix visible as a distinct event in history — useful when tracing production incidents later.
