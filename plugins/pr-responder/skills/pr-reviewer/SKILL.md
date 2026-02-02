---
name: pr-comment-analyzer
description: Analyzes PR review comments to determine actionability and priority. Use when reviewing GitHub PR comments or preparing to respond to code review feedback.
allowed-tools: Bash, Read, Grep, Glob, AskUserQuestion, Edit
---

# PR Comment Analysis Skill

When analyzing PR review comments, apply these principles:

## Comment Classification

### Actionable Comments (High Priority)
- Bug reports or potential issues
- Security concerns
- Performance problems
- Missing error handling
- Type safety issues
- Breaking changes

### Actionable Comments (Medium Priority)
- Code style violations
- Naming improvements
- Refactoring suggestions
- Documentation gaps
- Test coverage requests

### Non-Actionable Comments
- Praise or acknowledgments ("LGTM", "Nice!")
- Questions that need verbal response only
- Already resolved conversations
- Suggestions explicitly marked as optional/nitpick

## GitHub CLI Commands Reference

### Get PR for current branch
```bash
gh pr view --json number,title,url,state,baseRefName,headRefName
```

### Get review comments (inline comments on code)
```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments --jq '.[] | {id, path, line, body, user: .user.login, created_at, in_reply_to_id}'
```

### Get issue comments (general PR discussion)
```bash
gh api repos/{owner}/{repo}/issues/{pr_number}/comments --jq '.[] | {id, body, user: .user.login, created_at}'
```

### Get repo owner and name
```bash
gh repo view --json owner,name --jq '"\(.owner.login)/\(.name)"'
```

### Get review threads with resolution status
```bash
gh api graphql -f query='
  query($owner: String!, $repo: String!, $pr: Int!) {
    repository(owner: $owner, name: $repo) {
      pullRequest(number: $pr) {
        reviewThreads(first: 100) {
          nodes {
            isResolved
            comments(first: 10) {
              nodes {
                body
                author { login }
                path
                line
              }
            }
          }
        }
      }
    }
  }
' -f owner=OWNER -f repo=REPO -F pr=PR_NUMBER
```

## Response Strategy

When presenting comments to the user:

1. **Group by file** - Makes it easier to understand context
2. **Show severity** - Distinguish must-fix from nice-to-have
3. **Propose solutions** - Don't just show the problem
4. **Respect reviewer intent** - Understand what they're really asking for

## Implementation Guidelines

When implementing fixes:

1. **Read surrounding context** - Understand the code before changing it
2. **Match existing style** - Don't introduce inconsistencies
3. **Minimal changes** - Fix only what's requested
4. **Verify imports** - Add any needed imports for new code
5. **Consider tests** - Mention if tests might need updating
