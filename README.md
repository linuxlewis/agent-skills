# Claude Skills

Custom Claude Code plugins and skills marketplace.

## Installation

Add this marketplace to Claude Code:

```bash
claude plugin marketplace add linuxlewis/claude-skills
```

Then install plugins:

```bash
claude plugin install agent-browser@linuxlewis-skills
claude plugin install pr-responder@linuxlewis-skills
```

## Available Plugins

### agent-browser

Browser automation skill using the `agent-browser` CLI. Enables Claude to:

- Navigate websites
- Interact with page elements (click, type, fill)
- Take screenshots and PDFs
- Extract content from pages
- Handle forms and interactive elements

### pr-responder

Review and respond to PR comments automatically. Features:

- Fetches all PR comments for the current branch
- Categorizes comments by actionability (bugs, style, questions, etc.)
- Presents recommendations for which comments to address
- Implements approved changes automatically
- Summarizes changes made

**Command:** `/respond` - Review PR comments and implement approved changes

**Requirements:** GitHub CLI (`gh`) installed and authenticated

## Structure

```
claude-skills/
├── .claude-plugin/
│   └── marketplace.json
├── plugins/
│   ├── agent-browser/
│   │   ├── .claude-plugin/plugin.json
│   │   ├── skills/agent-browser/SKILL.md
│   │   └── README.md
│   └── pr-responder/
│       ├── .claude-plugin/plugin.json
│       ├── commands/respond.md
│       ├── skills/pr-reviewer/SKILL.md
│       └── README.md
└── README.md
```

## License

MIT
