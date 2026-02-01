---
name: agent-browser
description: This skill should be used when the user asks to "browse a website", "open a webpage", "interact with a page", "take a screenshot", "fill out a form", "click a button", "scrape content", "automate browser tasks", or needs to navigate and interact with web pages.
version: 1.0.0
---

# Agent Browser Skill

Browser automation using the `agent-browser` CLI - a fast, headless browser automation tool for AI agents.

## Installation

```bash
npm install -g agent-browser
# or use with npx
npx agent-browser install
```

## Quick Start

```bash
# Navigate to a URL
agent-browser open https://example.com

# Get accessibility snapshot (shows refs like @e1, @e2)
agent-browser snapshot -i

# Click using ref from snapshot
agent-browser click @e2

# Type into an element
agent-browser fill @e3 "hello world"

# Take screenshot
agent-browser screenshot output.png
```

## Core Commands

### Navigation
```bash
agent-browser open <url>           # Navigate to URL
agent-browser back                 # Go back
agent-browser forward              # Go forward
agent-browser reload               # Reload page
```

### Interaction
```bash
agent-browser click <sel>          # Click element (or @ref)
agent-browser dblclick <sel>       # Double-click
agent-browser type <sel> <text>    # Type into element
agent-browser fill <sel> <text>    # Clear and fill
agent-browser press <key>          # Press key (Enter, Tab, Control+a)
agent-browser hover <sel>          # Hover element
agent-browser focus <sel>          # Focus element
agent-browser check <sel>          # Check checkbox
agent-browser uncheck <sel>        # Uncheck checkbox
agent-browser select <sel> <val>   # Select dropdown option
```

### Getting Information
```bash
agent-browser snapshot             # Accessibility tree with refs
agent-browser snapshot -i          # Interactive elements only
agent-browser snapshot -c          # Compact (no empty elements)
agent-browser get text <sel>       # Get element text
agent-browser get html <sel>       # Get element HTML
agent-browser get value <sel>      # Get input value
agent-browser get title            # Get page title
agent-browser get url              # Get current URL
```

### Capture
```bash
agent-browser screenshot [path]    # Take screenshot
agent-browser screenshot --full    # Full page screenshot
agent-browser pdf <path>           # Save as PDF
```

### Waiting
```bash
agent-browser wait <sel>           # Wait for element
agent-browser wait 2000            # Wait milliseconds
```

## Workflow Pattern

The typical workflow for browser automation:

1. **Open** - Navigate to the target URL
2. **Snapshot** - Get the accessibility tree to see available elements
3. **Interact** - Use refs (@e1, @e2, etc.) to interact with elements
4. **Verify** - Take a snapshot or screenshot to verify state

### Example: Form Submission

```bash
# 1. Open the page
agent-browser open https://example.com/login

# 2. Get interactive elements
agent-browser snapshot -i
# Output shows: @e1 textbox "Email", @e2 textbox "Password", @e3 button "Sign In"

# 3. Fill the form
agent-browser fill @e1 "user@example.com"
agent-browser fill @e2 "password123"

# 4. Submit
agent-browser click @e3

# 5. Wait and verify
agent-browser wait 2000
agent-browser snapshot -i
```

### Example: Scraping Content

```bash
# Open page
agent-browser open https://news.ycombinator.com

# Get snapshot to see structure
agent-browser snapshot

# Get specific text content
agent-browser get text ".titleline"

# Screenshot for visual reference
agent-browser screenshot hn.png
```

## Sessions

Use sessions to maintain browser state across commands:

```bash
# Start a named session
AGENT_BROWSER_SESSION=myproject agent-browser open https://example.com

# Continue in same session
AGENT_BROWSER_SESSION=myproject agent-browser snapshot
AGENT_BROWSER_SESSION=myproject agent-browser click @e1

# Or use --session flag
agent-browser --session myproject open https://example.com
```

## Selectors

The tool supports multiple selector types:

- **Refs**: `@e1`, `@e2` (from snapshot output)
- **CSS**: `#id`, `.class`, `div > span`
- **Text**: `text=Submit`
- **Role**: `role=button[name="Submit"]`

## Options

```bash
--session <name>     # Isolated session name
--headed             # Show browser window (not headless)
--json               # JSON output
--full, -f           # Full page screenshot
--debug              # Debug output
```

## Environment Variables

```bash
AGENT_BROWSER_SESSION           # Default session name
AGENT_BROWSER_EXECUTABLE_PATH   # Custom browser path
AGENT_BROWSER_PROVIDER          # Cloud browser provider
```

## Best Practices

1. **Always snapshot first** - Get the accessibility tree before interacting
2. **Use refs** - Prefer `@e1` refs from snapshot over CSS selectors
3. **Use sessions** - Maintain state across multiple commands
4. **Wait appropriately** - Use `wait` for dynamic content
5. **Interactive flag** - Use `snapshot -i` to focus on actionable elements
6. **Verify actions** - Snapshot or screenshot after interactions to confirm state

## Troubleshooting

- **Browser not installed**: Run `agent-browser install`
- **Element not found**: Try `snapshot` to see available elements
- **Timing issues**: Add `wait` commands for dynamic pages
- **State lost**: Use `--session` to maintain browser state
