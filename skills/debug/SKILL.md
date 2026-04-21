---
name: debug
description: Enable debug logging for this session and help diagnose issues by reading the session debug log.
---

# Debug Skill

Help the user debug an issue they're encountering in this current Claude Code session.

## Session Debug Log

The debug log for the current session is at: `~/.claude/debug/<session-id>.txt`

### Reading the Log

The log contains all event logging for this session. Look for:
- `[ERROR]` entries — actual errors
- `[WARN]` entries — warnings that may indicate issues
- Stack traces — showing where errors occurred
- Tool invocations — what was attempted

### Debug Log Format

```
[2024-01-15T10:30:45.123Z] [INFO] Session started
[2024-01-15T10:31:02.456Z] [ERROR] Tool execution failed: ...
[2024-01-15T10:31:15.789Z] [WARN] Permission denied for: ...
```

## Instructions

1. Read the debug log file at `~/.claude/debug/` (find the current session file)
2. Search for `[ERROR]` and `[WARN]` lines across the full file
3. Explain what you found in plain language
4. Suggest concrete fixes or next steps

## Enabling Debug Logging

If debug logging wasn't enabled at session start, you can enable it now by running:

```bash
claude --debug
```

This will start a new session with debug logging enabled from the beginning.

## Common Issues to Look For

| Pattern | Likely Cause |
|---------|--------------|
| Permission denied | Missing permission rules in settings |
| Tool execution failed | Tool input validation error |
| MCP server timeout | Server not responding |
| Rate limit exceeded | API quota exhausted |
| Network error | Connectivity issues |

## Settings Files

Remember that settings are in:
- User settings: `~/.claude/settings.json`
- Project settings: `.claude/settings.json`
- Local settings: `.claude/settings.local.json`

## Usage

```
/debug                    # Read current session log and summarize errors
/debug permission issue   # Focus on permission-related issues
/debug MCP server error   # Focus on MCP server issues
```