---
name: openclaw-notify
description: Notify OpenClaw gateway when background tasks complete. Use when you were dispatched by OpenClaw/TARS for background work and need to ping the user that you're done, failed, or blocked.
license: MIT
compatibility: Requires openclaw CLI installed and gateway running
metadata:
  author: linuxlewis
  version: "1.0.0"
---

# OpenClaw Notify

Send notifications back to the OpenClaw session that dispatched you.

## The Command

```bash
openclaw gateway wake --text "Your message here" --mode now
```

That's it. One command.

## When to Use

Call this **once** at the very end of your task:

```bash
# ✅ Task completed
openclaw gateway wake --text "Done: Created PR #42 with OAuth implementation" --mode now

# ✅ Task failed
openclaw gateway wake --text "Failed: Could not push branch - permission denied" --mode now

# ✅ Blocked on something
openclaw gateway wake --text "Blocked: Need API credentials to continue" --mode now
```

**Don't** send multiple notifications:

```bash
# ❌ Unnecessary progress updates
openclaw gateway wake --text "Starting work..." --mode now
openclaw gateway wake --text "Halfway done..." --mode now
```

## Message Format

Keep it short and useful:

```
Done: <what you accomplished>
Failed: <what went wrong>
Blocked: <what you need>
```

Include links when relevant (PR URLs, file paths, error logs).

## Example Workflow

When dispatched with "implement feature X and make a PR":

```bash
# 1. Do your work
git checkout -b feature/x
# ... implement feature ...
git add -A && git commit -m "feat: implement X"
git push -u origin feature/x
gh pr create --title "feat: implement X" --body "..."

# 2. Notify when done (LAST step)
openclaw gateway wake --text "Done: PR #15 created - https://github.com/org/repo/pull/15" --mode now
```

## Troubleshooting

If the command fails:

```bash
# Check if openclaw is available
which openclaw

# Check gateway status  
openclaw gateway status
```

If gateway isn't running, the notification won't deliver — but that's okay, the user will check on you eventually.
