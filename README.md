# Claude Skills

Custom Claude Code plugins and skills.

## Installation

Add this marketplace to Claude Code:

```bash
claude plugin marketplace add linuxlewis/claude-skills
```

Then install skills:

```bash
claude plugin install agent-browser@linuxlewis-skills
```

## Available Skills

### agent-browser

Browser automation skill using the `agent-browser` CLI. Enables Claude to:

- Navigate websites
- Interact with page elements (click, type, fill)
- Take screenshots and PDFs
- Extract content from pages
- Handle forms and interactive elements

## Structure

```
claude-skills/
├── .claude-plugin/
│   └── plugin.json        # Plugin metadata
├── skills/
│   └── agent-browser/
│       └── SKILL.md       # Browser automation skill
└── README.md
```

## License

MIT
