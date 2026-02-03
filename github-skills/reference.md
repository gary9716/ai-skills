# GitHub CLI Reference

## Authentication

```bash
# Check auth status
gh auth status

# Login interactively
gh auth login

# Login with token
gh auth login --with-token < token.txt

# Switch accounts
gh auth switch

# Logout
gh auth logout
```

## Pull Request Commands

### Creating PRs

```bash
# Basic create
gh pr create --title "Title" --body "Description"

# Create draft PR
gh pr create --draft --title "WIP: Feature"

# Create with reviewers
gh pr create --reviewer user1,user2 --title "Title"

# Create with labels
gh pr create --label bug,priority --title "Fix bug"

# Create targeting specific base
gh pr create --base develop --title "Feature"

# Create from specific branch
gh pr create --head feature-branch --title "Title"

# Fill from commit messages
gh pr create --fill
```

### Viewing PRs

```bash
# View current branch's PR
gh pr view

# View specific PR
gh pr view 123

# View in browser
gh pr view 123 --web

# View as JSON
gh pr view 123 --json title,body,state,author

# View PR diff
gh pr diff 123

# View PR checks status
gh pr checks 123
```

### Listing PRs

```bash
# List open PRs
gh pr list

# List all PRs
gh pr list --state all

# List closed PRs
gh pr list --state closed

# List merged PRs
gh pr list --state merged

# List my PRs
gh pr list --author @me

# List PRs assigned to me
gh pr list --assignee @me

# List PRs for review
gh pr list --search "review-requested:@me"

# Limit results
gh pr list --limit 10

# JSON output
gh pr list --json number,title,author
```

### Managing PRs

```bash
# Checkout PR locally
gh pr checkout 123

# Close PR
gh pr close 123

# Reopen PR
gh pr reopen 123

# Merge PR
gh pr merge 123

# Merge with squash
gh pr merge 123 --squash

# Merge with rebase
gh pr merge 123 --rebase

# Delete branch after merge
gh pr merge 123 --delete-branch

# Mark as ready for review
gh pr ready 123

# Convert to draft
gh pr ready 123 --undo
```

### PR Reviews

```bash
# Approve PR
gh pr review 123 --approve

# Approve with comment
gh pr review 123 --approve --body "LGTM!"

# Request changes
gh pr review 123 --request-changes --body "Please fix X"

# Comment without approval
gh pr review 123 --comment --body "Question about line 42"
```

### PR Comments

```bash
# Add comment
gh pr comment 123 --body "Comment text"

# Add comment from file
gh pr comment 123 --body-file comment.md

# Edit last comment
gh pr comment 123 --edit-last --body "Updated comment"
```

## Repository Commands

```bash
# View repo
gh repo view

# View in browser
gh repo view --web

# Clone repo
gh repo clone owner/repo

# Fork repo
gh repo fork owner/repo

# Create repo
gh repo create my-repo --public

# List repos
gh repo list
```

## Issue Commands

```bash
# Create issue
gh issue create --title "Bug" --body "Description"

# View issue
gh issue view 123

# List issues
gh issue list

# Close issue
gh issue close 123

# Comment on issue
gh issue comment 123 --body "Comment"
```

## API Commands

```bash
# GET request
gh api repos/{owner}/{repo}

# POST request
gh api repos/{owner}/{repo}/issues --method POST -f title="Bug" -f body="Description"

# With pagination
gh api repos/{owner}/{repo}/pulls --paginate

# With jq filtering
gh api repos/{owner}/{repo}/pulls --jq '.[].title'

# GraphQL query
gh api graphql -f query='{ viewer { login }}'
```

## Workflow Commands

```bash
# List workflows
gh workflow list

# View workflow runs
gh run list

# View specific run
gh run view 12345

# Watch run in progress
gh run watch 12345

# Rerun failed jobs
gh run rerun 12345 --failed

# Download artifacts
gh run download 12345
```

## Useful Patterns

### Get PR URL after creation
```bash
PR_URL=$(gh pr create --title "Title" --body "Body" 2>&1)
echo "Created: $PR_URL"
```

### Check if PR exists for current branch
```bash
gh pr view --json number 2>/dev/null && echo "PR exists" || echo "No PR"
```

### Get PR number for current branch
```bash
gh pr view --json number --jq '.number'
```

### List files changed in PR
```bash
gh pr diff 123 --name-only
```

### Get PR review status
```bash
gh pr view 123 --json reviewDecision --jq '.reviewDecision'
```

### Check CI status
```bash
gh pr checks 123 --watch
```
