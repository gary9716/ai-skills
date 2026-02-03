---
name: github
description: Use when the user wants to interact with GitHub - create PRs, read PRs, comment on PRs, push commits, or pull commits. Triggers on keywords like "create PR", "pull request", "push to remote", "pull from remote", "PR comment", "review PR".
argument-hint: <action> [args] - e.g., "create-pr", "read-pr 123", "comment-pr 123", "push", "pull"
user-invocable: true
allowed-tools: Bash(gh *), Bash(git push*), Bash(git pull*), Bash(git status*), Bash(git log*), Bash(git diff*), Bash(git branch*), Bash(git remote*), Read, Grep, Glob
---

# GitHub Operations Skill

This skill provides GitHub operations using the `gh` CLI tool.

## Prerequisites

Ensure the user has `gh` CLI installed and authenticated:
```bash
gh auth status
```

If not authenticated, inform the user to run `gh auth login`.

## Available Operations

### 1. Create Pull Request (`create-pr`)

Create a new pull request from the current branch.

**Steps:**
1. Check current branch and ensure it's not the main/master branch
2. Check for uncommitted changes and warn the user
3. Verify the branch has been pushed to remote
4. Gather PR information (title, body, base branch)
5. Create the PR using `gh pr create`

**Command format:**
```bash
# Check current state
git status
git branch --show-current
git log origin/main..HEAD --oneline  # or origin/master

# Push branch if needed
git push -u origin <branch-name>

# Create PR
gh pr create --title "PR Title" --body "$(cat <<'EOF'
## Summary
- Description of changes

## Test Plan
- How to test

EOF
)"
```

**Options:**
- `--base <branch>` - Target branch (default: main/master)
- `--draft` - Create as draft PR
- `--assignee @me` - Assign to yourself
- `--reviewer <user>` - Request review

### 2. Read Pull Request (`read-pr`)

View details of a pull request.

**Command format:**
```bash
# View PR details
gh pr view <number>

# View PR in web browser
gh pr view <number> --web

# View PR diff
gh pr diff <number>

# List PR files
gh pr view <number> --json files --jq '.files[].path'

# View PR comments
gh api repos/{owner}/{repo}/pulls/<number>/comments

# View PR reviews
gh pr view <number> --json reviews
```

**List PRs:**
```bash
# List open PRs
gh pr list

# List PRs by author
gh pr list --author @me

# List PRs with specific state
gh pr list --state all
```

### 3. Comment on Pull Request (`comment-pr`)

Add a comment to a pull request.

**Command format:**
```bash
# Add a general comment
gh pr comment <number> --body "Your comment here"

# Add a review comment
gh pr review <number> --comment --body "Review comment"

# Approve PR
gh pr review <number> --approve --body "LGTM!"

# Request changes
gh pr review <number> --request-changes --body "Please fix..."
```

### 4. Push Commits (`push`)

Push local commits to the remote repository.

**Steps:**
1. Check current branch
2. Check for uncommitted changes
3. Verify remote tracking branch
4. Push commits

**Command format:**
```bash
# Check status first
git status
git branch -vv  # Show tracking info

# Push to remote
git push

# Push and set upstream (for new branches)
git push -u origin <branch-name>

# Push specific branch
git push origin <branch-name>
```

**Safety checks:**
- Never force push to main/master without explicit user confirmation
- Warn if there are uncommitted changes

### 5. Pull Commits (`pull`)

Pull commits from the remote repository.

**Steps:**
1. Check for uncommitted changes (warn if any)
2. Pull from remote

**Command format:**
```bash
# Check status first
git status

# Pull with rebase (preferred)
git pull --rebase

# Pull with merge
git pull

# Pull specific branch
git pull origin <branch-name>
```

## Argument Parsing

Parse the first argument to determine the operation:

| Argument | Operation |
|----------|-----------|
| `create-pr`, `create`, `new-pr` | Create Pull Request |
| `read-pr`, `view-pr`, `show-pr`, `pr` | Read Pull Request |
| `comment-pr`, `comment`, `review` | Comment on Pull Request |
| `push` | Push Commits |
| `pull` | Pull Commits |

If a PR number follows the action (e.g., `read-pr 123`), use it as the target PR.

## Error Handling

1. **Not a git repository**: Inform user they need to be in a git repo
2. **gh not installed**: Provide installation instructions (`brew install gh` or visit https://cli.github.com)
3. **Not authenticated**: Guide user to run `gh auth login`
4. **No remote configured**: Help user add a remote with `git remote add origin <url>`
5. **Branch not pushed**: Offer to push the branch first

## Output Format

After completing an operation, provide:
1. A brief summary of what was done
2. Relevant links (PR URL, etc.)
3. Next steps if applicable

## Examples

**Create a PR:**
```
User: /github create-pr
Assistant: [Checks branch, creates PR, returns URL]
```

**Read PR #42:**
```
User: /github read-pr 42
Assistant: [Shows PR title, description, status, reviewers]
```

**Comment on PR:**
```
User: /github comment-pr 42 "Looks good, just one small suggestion..."
Assistant: [Adds comment, confirms]
```

**Push changes:**
```
User: /github push
Assistant: [Checks status, pushes, shows result]
```

**Pull latest:**
```
User: /github pull
Assistant: [Pulls with rebase, shows updated commits]
```
