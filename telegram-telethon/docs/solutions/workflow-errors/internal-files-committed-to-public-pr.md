---
title: Internal todo files accidentally committed to public PR
category: workflow-errors
tags:
  - git
  - gitignore
  - code-review
  - internal-files
  - pr-hygiene
components:
  - git
  - version-control
severity: medium
date_solved: 2026-01-18
---

# Internal Files Accidentally Committed to Public PR

## Problem

Internal development tracking files (`todos/` directory containing code review findings) were accidentally committed and pushed to a public GitHub PR, exposing internal development artifacts.

### Symptoms

- PR contained 15+ todo files with internal code review notes
- Files like `010-complete-p1-missing-error-handling.md` visible in public PR
- Internal process documentation exposed publicly

## Root Cause

1. No `.gitignore` rule existed for the `todos/` directory
2. Broad `git add` command staged all files including internal tracking files
3. Commit was pushed before noticing unwanted files were included

## Solution

### Step 1: Undo the commit while keeping changes staged

```bash
git reset HEAD~1 --soft
```

The `--soft` flag preserves your work in the staging area.

### Step 2: Unstage the accidentally committed directory

```bash
git reset HEAD todos/
```

Removes `todos/` from staging while keeping other files staged.

### Step 3: Recommit without the internal files

```bash
git commit -m "Your commit message"
```

### Step 4: Force push to update the remote branch

```bash
git push --force-with-lease
```

Using `--force-with-lease` is safer than `--force` - it fails if someone else pushed to the branch.

### Step 5: Add .gitignore to prevent future accidents

```gitignore
# Internal directories
todos/
plans/

# Build artifacts
*.egg-info/
__pycache__/
.venv/

# IDE
.idea/
.vscode/
```

Then commit the `.gitignore`:

```bash
git add .gitignore
git commit -m "chore: Add .gitignore"
git push
```

## Prevention Strategies

### 1. Always have .gitignore first

Before creating internal directories, add them to `.gitignore`:

```bash
echo "todos/" >> .gitignore
echo "plans/" >> .gitignore
git add .gitignore
git commit -m "Add gitignore for internal directories"
```

### 2. Review staged files before committing

```bash
git status              # See what's staged
git diff --cached       # See actual changes
```

### 3. Use pre-commit hook

Create `.git/hooks/pre-commit`:

```bash
#!/bin/bash
FORBIDDEN="todos/ plans/ _internal/"
STAGED=$(git diff --cached --name-only)

for pattern in $FORBIDDEN; do
    if echo "$STAGED" | grep -q "^$pattern"; then
        echo "ERROR: Internal files staged: $pattern"
        exit 1
    fi
done
```

Make executable: `chmod +x .git/hooks/pre-commit`

### 4. Use naming conventions

Prefix internal directories with `_` and ignore by pattern:

```gitignore
_*/
```

## Key Commands Reference

| Command | Purpose |
|---------|---------|
| `git reset HEAD~1 --soft` | Undo last commit, keep changes staged |
| `git reset HEAD <path>` | Unstage specific file/directory |
| `git push --force-with-lease` | Safe force push |
| `git rm -r --cached <path>` | Remove from git but keep local files |

## Related

- [Git documentation on reset](https://git-scm.com/docs/git-reset)
- [Pre-commit framework](https://pre-commit.com/)
