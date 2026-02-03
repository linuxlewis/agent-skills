---
name: pr-responder
description: Review and respond to GitHub PR comments. Use when analyzing PR feedback, determining which comments to address, classifying review comments by priority, or implementing fixes for code review feedback.
license: MIT
compatibility: Requires GitHub CLI (gh) installed and authenticated
metadata:
  author: linuxlewis
  version: "1.0.0"
allowed-tools: Bash Read Grep Glob Edit
---

# PR Responder

Analyze and respond to GitHub PR review comments efficiently.

## Workflow

1. **Find PR** - Get the PR for the current branch
2. **Fetch Comments** - Get all review comments
3. **Classify** - Categorize by actionability
4. **Present** - Show recommendations to user
5. **Implement** - Apply approved changes
6. **Summarize** - Report what was done

## Quick Start

```bash
# Get PR for current branch
gh pr view --json number,title,url,state

# Get review comments
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments
```

## Comment Classification

### High Priority (Must Fix)
- Bug reports or potential issues
- Security concerns
- Performance problems
- Missing error handling
- Type safety issues
- Breaking changes

### Medium Priority (Should Fix)
- Code style violations
- Naming improvements
- Refactoring suggestions
- Documentation gaps
- Test coverage requests

### Low Priority (Optional)
- Praise or acknowledgments ("LGTM", "Nice!")
- Questions needing verbal response only
- Already resolved conversations
- Suggestions marked as optional/nitpick

## Implementation Guidelines

1. **Read surrounding context** - Understand the code before changing
2. **Match existing style** - Don't introduce inconsistencies
3. **Minimal changes** - Fix only what's requested
4. **Verify imports** - Add any needed imports
5. **Consider tests** - Note if tests need updating

See [references/github-api.md](references/github-api.md) for complete API reference.
