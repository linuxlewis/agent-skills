# Agent Browser Command Reference

Complete reference for all agent-browser CLI commands.

## Navigation

```bash
agent-browser open <url>           # Navigate to URL
agent-browser back                 # Go back
agent-browser forward              # Go forward
agent-browser reload               # Reload page
```

## Interaction

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
agent-browser drag <src> <dst>     # Drag and drop
agent-browser upload <sel> <files> # Upload files
```

## Getting Information

```bash
agent-browser snapshot             # Accessibility tree with refs
agent-browser snapshot -i          # Interactive elements only
agent-browser snapshot -c          # Compact (no empty elements)
agent-browser snapshot -d <n>      # Limit tree depth
agent-browser get text <sel>       # Get element text
agent-browser get html <sel>       # Get element HTML
agent-browser get value <sel>      # Get input value
agent-browser get title            # Get page title
agent-browser get url              # Get current URL
agent-browser get count <sel>      # Count matching elements
agent-browser get box <sel>        # Get bounding box
```

## State Checking

```bash
agent-browser is visible <sel>     # Check if visible
agent-browser is enabled <sel>     # Check if enabled
agent-browser is checked <sel>     # Check if checked
```

## Capture

```bash
agent-browser screenshot [path]    # Take screenshot
agent-browser screenshot --full    # Full page screenshot
agent-browser pdf <path>           # Save as PDF
```

## Waiting

```bash
agent-browser wait <sel>           # Wait for element
agent-browser wait 2000            # Wait milliseconds
agent-browser scrollintoview <sel> # Scroll element into view
```

## Options

```bash
--session <name>     # Isolated session name
--headed             # Show browser window (not headless)
--json               # JSON output
--full, -f           # Full page screenshot
--debug              # Debug output
--profile <path>     # Persistent browser profile
```

## Environment Variables

```bash
AGENT_BROWSER_SESSION           # Default session name
AGENT_BROWSER_EXECUTABLE_PATH   # Custom browser path
AGENT_BROWSER_PROVIDER          # Cloud browser provider
AGENT_BROWSER_PROXY             # Proxy server URL
```

## Examples

### Form Submission

```bash
agent-browser open https://example.com/login
agent-browser snapshot -i
# Output: @e1 textbox "Email", @e2 textbox "Password", @e3 button "Sign In"
agent-browser fill @e1 "user@example.com"
agent-browser fill @e2 "password123"
agent-browser click @e3
agent-browser wait 2000
agent-browser snapshot -i
```

### Scraping Content

```bash
agent-browser open https://news.ycombinator.com
agent-browser snapshot
agent-browser get text ".titleline"
agent-browser screenshot hn.png
```

### With Sessions

```bash
agent-browser --session shop open https://shop.example.com
agent-browser --session shop click "text=Add to Cart"
agent-browser --session shop click "text=Checkout"
```
