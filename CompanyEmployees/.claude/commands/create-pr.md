# Git and Pull Request Commands

## Check Current Status
```bash
# View current branch and changes
git status

# View uncommitted changes
git diff

# View staged changes
git diff --staged

# View recent commits
git log --oneline -10
```

## Create and Switch Branches
```bash
# Create new branch from current branch
git checkout -b feature/your-feature-name

# Create new branch from main/master
git checkout -b feature/your-feature-name origin/main

# Switch to existing branch
git checkout branch-name

# List all branches
git branch -a
```

## Stage and Commit Changes
```bash
# Stage all changes
git add .

# Stage specific files
git add file1.cs file2.cs

# Stage by pattern
git add "*.cs"

# Commit with message
git commit -m "feat: add new feature description"

# Amend last commit (add changes to previous commit)
git add .
git commit --amend

# Commit with multi-line message
git commit -m "feat: add new feature" -m "- Added X functionality" -m "- Updated Y component"
```

## Push Changes
```bash
# Push new branch to remote
git push -u origin feature/your-feature-name

# Push to existing remote branch
git push

# Force push (use carefully!)
git push --force-with-lease
```

## Create Pull Request with GitHub CLI
```bash
# Create PR interactively
gh pr create

# Create PR with title and body
gh pr create --title "feat: add new feature" --body "Description of changes"

# Create PR with template
gh pr create --title "feat: add new feature" --body "$(cat <<'EOF'
## Summary
- Added new feature X
- Updated component Y
- Fixed bug Z

## Test Plan
- [ ] Unit tests pass
- [ ] Manual testing completed
- [ ] No console errors

## Screenshots
(if applicable)
EOF
)"

# Create draft PR
gh pr create --draft --title "WIP: feature name"

# Create PR and assign reviewers
gh pr create --title "feat: add feature" --reviewer username1,username2

# Create PR with labels
gh pr create --title "feat: add feature" --label "enhancement,needs-review"
```

## View and Manage PRs
```bash
# List PRs
gh pr list

# View PR details
gh pr view 123

# View PR in browser
gh pr view 123 --web

# Check PR status
gh pr status

# Checkout a PR locally
gh pr checkout 123

# Close PR
gh pr close 123

# Reopen PR
gh pr reopen 123

# Merge PR
gh pr merge 123
```

## Sync with Remote
```bash
# Fetch latest changes
git fetch origin

# Pull latest changes from current branch
git pull

# Pull and rebase
git pull --rebase

# Update branch with main
git checkout main
git pull
git checkout feature/your-branch
git merge main
# OR
git rebase main
```

## Clean Up
```bash
# Delete local branch
git branch -d branch-name

# Force delete local branch
git branch -D branch-name

# Delete remote branch
git push origin --delete branch-name

# Prune deleted remote branches
git fetch --prune
```

## Useful Aliases (add to ~/.gitconfig)
```bash
[alias]
    st = status
    co = checkout
    br = branch
    ci = commit
    pl = pull
    ps = push
    lg = log --oneline --graph --decorate
    unstage = reset HEAD --
    last = log -1 HEAD
    pr = !gh pr create
```

## Common PR Workflow
```bash
# 1. Create feature branch
git checkout -b feature/new-feature

# 2. Make changes and commit
git add .
git commit -m "feat: implement new feature"

# 3. Push to remote
git push -u origin feature/new-feature

# 4. Create PR
gh pr create --title "feat: implement new feature" --body "$(cat <<'EOF'
## Summary
Description of what this PR does

## Changes
- Change 1
- Change 2

## Testing
- [ ] Unit tests pass
- [ ] Manual testing done
EOF
)"

# 5. Address review comments
git add .
git commit -m "fix: address review comments"
git push

# 6. After approval, merge (or let reviewer merge)
gh pr merge --squash
```

## Tips
- Always pull latest changes before creating a new branch
- Write clear, descriptive commit messages
- Keep PRs focused and small when possible
- Test your changes before creating PR
- Use draft PRs for work in progress
- Link related issues in PR description using #issue-number
- Run linters and tests before pushing