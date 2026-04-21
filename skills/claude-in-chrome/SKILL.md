---
name: claude-in-chrome
description: Automates your Chrome browser to interact with web pages - clicking elements, filling forms, capturing screenshots, reading console logs, and navigating sites. Opens pages in new tabs within your existing Chrome session.
---

# Claude in Chrome: Browser Automation

Automate your Chrome browser to interact with web pages.

## What This Skill Does

- Click elements on web pages
- Fill forms and input fields
- Capture screenshots
- Read console logs
- Navigate between pages
- Execute JavaScript in browser context

## Prerequisites

- Chrome browser must be installed
- Claude for Chrome extension must be installed and enabled
- Site-level permissions must be configured in the extension

## Available Tools

When this skill is activated, you have access to Chrome browser automation tools:

| Tool | Purpose |
|------|---------|
| `mcp__claude-in-chrome__tabs_context` | Get current browser tab information |
| `mcp__claude-in-chrome__navigate` | Navigate to a URL |
| `mcp__claude-in-chrome__click` | Click an element |
| `mcp__claude-in-chrome__type` | Type text into an element |
| `mcp__claude-in-chrome__screenshot` | Capture a screenshot |
| `mcp__claude-in-chrome__console_messages` | Read browser console |
| `mcp__claude-in-chrome__evaluate` | Execute JavaScript |

## Workflow

1. **Start by getting tab context** — Use `tabs_context` to see current browser state
2. **Navigate to target page** — Use `navigate` to open the desired URL
3. **Interact with elements** — Use `click`, `type`, or `evaluate` to interact
4. **Capture results** — Use `screenshot` or `console_messages` to verify actions

## Important Notes

- Opens pages in new tabs within your existing Chrome session
- Requires site-level permissions before executing (configured in the extension)
- Works with your actual Chrome browser, not a headless instance

## Usage

```
/claude-in-chrome                     # Activate browser automation
/claude-in-chrome navigate to example.com
/claude-in-chrome click the submit button
/claude-in-chrome screenshot the page
```