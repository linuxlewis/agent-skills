---
description: Review PR comments for the current branch and implement approved changes
---

# PR Comment Responder

You are helping the user respond to PR review comments. Follow these steps carefully:

## Step 1: Find the PR for the Current Branch

Run `gh pr view --json number,title,url,state` to find the PR associated with the current branch.

If no PR exists, inform the user and stop.

## Step 2: Fetch All PR Comments

Fetch all review comments using:
```
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments
```

Also fetch general PR comments:
```
gh api repos/{owner}/{repo}/issues/{pr_number}/comments
```

Parse the JSON response to extract:
- Comment author
- Comment body
- File path and line number (for review comments)
- Whether the comment is part of a review thread
- Whether the comment has been resolved

## Step 3: Analyze and Categorize Comments

For each unresolved comment, categorize it as:

1. **Actionable** - Requires code changes (bugs, improvements, style fixes)
2. **Question** - Needs a response but no code change
3. **Discussion** - Open-ended discussion or suggestion
4. **Resolved/Acknowledged** - Already addressed or just praise

Focus on **Actionable** comments.

## Step 4: Present Recommendations

Use the AskUserQuestion tool to present the actionable comments to the user.

For each actionable comment, show:
- The file and line number
- The reviewer's comment
- Your recommended action

Ask the user which comments they want to address. Allow multi-select.

Example question format:
```
Which PR comments would you like me to address?

Options:
1. [file.ts:42] "Consider using const here" - Change let to const
2. [api.ts:105] "Add error handling" - Wrap in try/catch
3. [utils.ts:23] "This could be simplified" - Refactor to use reduce()
```

## Step 5: Implement Approved Changes

For each comment the user approves:

1. Read the relevant file
2. Understand the context around the comment
3. Make the appropriate code change using the Edit tool
4. Verify the change doesn't break anything obvious

After implementing each change, briefly confirm what was done.

## Step 6: Summary

After all changes are implemented, provide a summary:
- Number of comments addressed
- Files modified
- Suggest the user run tests if applicable
- Remind them to push changes and mark conversations as resolved on GitHub

## Important Notes

- Never make changes the user hasn't approved
- If a comment is ambiguous, ask for clarification
- Preserve existing code style and formatting
- If a suggested change would break something, explain why and propose an alternative
