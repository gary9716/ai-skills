# GitHub Skill Examples

## Example 1: Complete PR Creation Workflow

**User Request:** "Create a PR for my current changes"

**Execution:**

```bash
# 1. Check current state
$ git status
On branch feature/add-auth
Changes not staged for commit:
  modified:   src/auth.ts

$ git branch --show-current
feature/add-auth

# 2. Commit changes (if needed)
$ git add src/auth.ts
$ git commit -m "Add authentication middleware"

# 3. Check if branch is pushed
$ git push -u origin feature/add-auth
Enumerating objects: 5, done.
...
To github.com:user/repo.git
 * [new branch]      feature/add-auth -> feature/add-auth

# 4. Check commits to include
$ git log origin/main..HEAD --oneline
abc1234 Add authentication middleware
def5678 Add auth types

# 5. Create the PR
$ gh pr create --title "Add authentication middleware" --body "$(cat <<'EOF'
## Summary
- Added JWT authentication middleware
- Integrated with existing user service

## Test Plan
- Run `npm test` to verify all tests pass
- Manual testing with Postman collection

EOF
)"

Creating pull request for feature/add-auth into main in user/repo
https://github.com/user/repo/pull/42
```

**Response:** "Created PR #42: https://github.com/user/repo/pull/42"

---

## Example 2: Read and Summarize a PR

**User Request:** "/github read-pr 42"

**Execution:**

```bash
$ gh pr view 42 --json title,body,state,author,reviewDecision,additions,deletions,changedFiles

{
  "title": "Add authentication middleware",
  "body": "## Summary\n- Added JWT authentication...",
  "state": "OPEN",
  "author": {"login": "developer"},
  "reviewDecision": "REVIEW_REQUIRED",
  "additions": 150,
  "deletions": 20,
  "changedFiles": 5
}

$ gh pr view 42 --json files --jq '.files[].path'
src/auth.ts
src/middleware/jwt.ts
src/types/auth.d.ts
tests/auth.test.ts
package.json
```

**Response:**
```
PR #42: Add authentication middleware

Author: @developer
State: Open (Review Required)
Changes: +150 / -20 across 5 files

Files changed:
- src/auth.ts
- src/middleware/jwt.ts
- src/types/auth.d.ts
- tests/auth.test.ts
- package.json

Summary:
- Added JWT authentication middleware
- Integrated with existing user service
```

---

## Example 3: Comment on a PR

**User Request:** "/github comment-pr 42 The implementation looks good, but please add error handling for expired tokens"

**Execution:**

```bash
$ gh pr comment 42 --body "The implementation looks good, but please add error handling for expired tokens"

https://github.com/user/repo/pull/42#issuecomment-123456
```

**Response:** "Comment added to PR #42"

---

## Example 4: Review and Approve a PR

**User Request:** "/github review 42 approve"

**Execution:**

```bash
# First, check the PR status
$ gh pr view 42 --json reviewDecision,reviews
{
  "reviewDecision": "REVIEW_REQUIRED",
  "reviews": []
}

# Approve the PR
$ gh pr review 42 --approve --body "LGTM! Code looks clean and tests are comprehensive."

Approved pull request #42
```

**Response:** "PR #42 approved with comment: 'LGTM! Code looks clean and tests are comprehensive.'"

---

## Example 5: Push Changes to Remote

**User Request:** "/github push"

**Execution:**

```bash
# Check status
$ git status
On branch feature/update-docs
Your branch is ahead of 'origin/feature/update-docs' by 2 commits.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean

# Check what will be pushed
$ git log origin/feature/update-docs..HEAD --oneline
abc1234 Update API documentation
def5678 Fix typos in README

# Push
$ git push
Enumerating objects: 8, done.
Counting objects: 100% (8/8), done.
Delta compression using up to 8 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (6/6), 1.2 KiB | 1.2 MiB/s, done.
Total 6 (delta 2), reused 0 (delta 0)
To github.com:user/repo.git
   old1234..abc1234  feature/update-docs -> feature/update-docs
```

**Response:** "Pushed 2 commits to origin/feature/update-docs"

---

## Example 6: Pull Latest Changes

**User Request:** "/github pull"

**Execution:**

```bash
# Check for local changes
$ git status
On branch main
Your branch is behind 'origin/main' by 3 commits, and can be fast-forwarded.

nothing to commit, working tree clean

# Pull with rebase
$ git pull --rebase
Updating abc1234..xyz9876
Fast-forward
 src/api.ts     | 25 +++++++++++++++++++++++++
 src/utils.ts   | 10 +++++-----
 package.json   |  2 +-
 3 files changed, 31 insertions(+), 6 deletions(-)
```

**Response:**
```
Pulled 3 commits from origin/main:
- xyz9876: Fix API rate limiting
- def5678: Update dependencies
- abc1234: Refactor utils

Files updated: src/api.ts, src/utils.ts, package.json
```

---

## Example 7: Create Draft PR

**User Request:** "Create a draft PR for my work in progress"

**Execution:**

```bash
$ git push -u origin feature/wip-refactor

$ gh pr create --draft --title "WIP: Refactor authentication system" --body "$(cat <<'EOF'
## Work in Progress

This PR refactors the authentication system. Still working on:
- [ ] Update tests
- [ ] Add migration script
- [ ] Update documentation

EOF
)"

Creating pull request for feature/wip-refactor into main in user/repo
https://github.com/user/repo/pull/43
```

**Response:** "Created draft PR #43: https://github.com/user/repo/pull/43"

---

## Example 8: List and Check PR Status

**User Request:** "Show me my open PRs"

**Execution:**

```bash
$ gh pr list --author @me --json number,title,state,reviewDecision,createdAt

[
  {
    "number": 42,
    "title": "Add authentication middleware",
    "state": "OPEN",
    "reviewDecision": "APPROVED",
    "createdAt": "2024-01-15T10:00:00Z"
  },
  {
    "number": 38,
    "title": "Fix login bug",
    "state": "OPEN",
    "reviewDecision": "CHANGES_REQUESTED",
    "createdAt": "2024-01-10T14:30:00Z"
  }
]
```

**Response:**
```
Your open PRs:

#42 - Add authentication middleware
     Status: Approved, ready to merge
     Created: Jan 15, 2024

#38 - Fix login bug
     Status: Changes requested
     Created: Jan 10, 2024
```

---

## Example 9: Handle Errors Gracefully

**User Request:** "/github create-pr" (but not in a git repo)

**Execution:**

```bash
$ git status
fatal: not a git repository (or any of the parent directories): .git
```

**Response:** "This directory is not a git repository. Please navigate to a git repository or initialize one with `git init`."

---

**User Request:** "/github push" (gh not authenticated)

**Execution:**

```bash
$ gh auth status
You are not logged into any GitHub hosts. Run gh auth login to authenticate.
```

**Response:** "You're not authenticated with GitHub. Please run `gh auth login` to authenticate, then try again."
