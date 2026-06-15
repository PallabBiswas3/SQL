# Git Commands Cheat Sheet (Advanced Interview Topics)

## 1. Internal Mechanics

```bash
# View object type/content in Git's internal database
git cat-file -p <hash>
git cat-file -t <hash>

# Check current HEAD status
git status

# Detached HEAD - create a new branch from current commit
git checkout -b <new-branch-name>

# View commit objects, trees, blobs
git log --oneline --graph --all
```

## 2. Collaboration & Branching

```bash
# Rebase current branch onto another
git rebase <branch>

# Merge another branch into current branch
git merge <branch>

# Interactive rebase to squash commits
git rebase -i HEAD~<number-of-commits>

# Trunk-based workflow - frequent small commits to main
git checkout main
git pull origin main
git add .
git commit -m "small change"
git push origin main
```

## 3. Large Files & ML/Data Workflows

```bash
# Initialize Git LFS
git lfs install

# Track large files with Git LFS
git lfs track "*.h5"
git lfs track "*.csv"

# Add and commit LFS-tracked files
git add .gitattributes
git add largefile.h5
git commit -m "Add large model file via LFS"

# DVC (Data Version Control) basics
dvc init
dvc add data/dataset.csv
git add data/dataset.csv.dvc .gitignore
git commit -m "Track dataset with DVC"
dvc push

# Diffing Jupyter notebooks
nbdime diff notebook1.ipynb notebook2.ipynb

# Convert notebook to script for cleaner diffs
jupytext --to py notebook.ipynb
```

## 4. Disaster Recovery

```bash
# Secret committed - rewrite history to remove it (after rotating the key)
git filter-repo --path path/to/secret_file --invert-paths

# Alternative: BFG Repo-Cleaner
bfg --delete-files secret_file.txt

# Force push cleaned history (after coordination with team)
git push origin --force --all

# Undo a commit already pushed (safe for shared branches)
git revert <commit-hash>

# Recover a deleted branch
git reflog
git checkout -b <recovered-branch> <commit-hash>

# Find the commit that introduced a bug
git bisect start
git bisect bad <bad-commit>
git bisect good <good-commit>
# Git checks out commits for testing; mark each as:
git bisect good
git bisect bad
# When done:
git bisect reset
```

## Quick Reference Table

| Scenario | Command |
|---|---|
| Inspect object internals | `git cat-file -p <hash>` |
| Fix detached HEAD | `git checkout -b <branch>` |
| Squash commits | `git rebase -i HEAD~N` |
| Merge vs rebase | `git merge` / `git rebase` |
| Track large files | `git lfs track "*.ext"` |
| Diff notebooks | `nbdime diff` |
| Remove secret from history | `git filter-repo` / `bfg` |
| Undo pushed commit | `git revert <hash>` |
| Recover deleted branch | `git reflog` |
| Find bug-introducing commit | `git bisect` |
