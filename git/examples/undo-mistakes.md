# Example — Undoing Mistakes

> Common scenarios and how to recover from them.

---

## Committed to the wrong branch

```bash
# Copy the commit to the right branch
git log --oneline          # note the commit hash
git switch correct-branch
git cherry-pick <hash>

# Remove it from the wrong branch
git switch wrong-branch
git reset --hard HEAD~1    # only if not pushed yet
```

---

## Pushed a bad commit

```bash
# Don't reset — revert instead (preserves history)
git revert <hash>
git push origin main
```

---

## Accidentally deleted a branch

```bash
git reflog                 # find the last commit hash of the branch
git switch -c recovered-branch <hash>
```

---

## Staged the wrong file

```bash
git restore --staged <file>   # unstage without losing changes
```

---

## Committed a secret / large file

```bash
# Remove from last commit (not yet pushed)
git reset --soft HEAD~1
git restore --staged <secret-file>
echo "<secret-file>" >> .gitignore
git add .gitignore
git commit -m "..."
```

If already pushed, use `git filter-repo` to rewrite history across all commits.

```bash
# Install (if needed)
pip install git-filter-repo

# Remove a file from the entire history
git filter-repo --path <secret-file> --invert-paths

# Remove a folder
git filter-repo --path secrets/ --invert-paths

# Force push rewritten history
git push origin --force --all
git push origin --force --tags
```

> **Always rotate the secret immediately** — assume it was already read. Rewriting history does not guarantee it was never cached (forks, CI logs, GitHub notifications).

> `git filter-repo` replaces the older `git filter-branch` — faster and safer.

---

## Lost changes after `--hard` reset

```bash
git reflog                        # find the hash before the reset
git reset --hard HEAD@{1}         # go back
```
