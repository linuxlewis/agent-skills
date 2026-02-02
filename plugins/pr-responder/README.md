# PR Responder Plugin

Review and respond to PR comments automatically. Analyzes PR feedback, recommends which comments to address, and implements approved changes.

## Features

- Fetches all PR comments for the current branch
- Categorizes comments by actionability
- Presents recommendations for which comments to address
- Implements approved changes automatically
- Summarizes changes made

## Commands

### `/respond`

Review PR comments for the current branch and implement approved changes.

## Skills

### pr-comment-analyzer

Automatically activated when reviewing GitHub PR comments. Helps classify comments by priority and suggests implementation strategies.

## Requirements

- GitHub CLI (`gh`) installed and authenticated
- Must be run from within a git repository with an open PR

## Usage

```bash
# In Claude Code, from a branch with an open PR:
/respond
```

Claude will:
1. Find the PR for your current branch
2. Fetch all review comments
3. Categorize them by actionability
4. Ask which ones you want to address
5. Implement the approved changes
6. Provide a summary
