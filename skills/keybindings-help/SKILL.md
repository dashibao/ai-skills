---
name: keybindings-help
description: Use when the user wants to customize keyboard shortcuts, rebind keys, add chord bindings, or modify ~/.claude/keybindings.json. Examples: "rebind ctrl+s", "add a chord shortcut", "change the submit key", "customize keybindings".
---

# Keybindings Skill

Create or modify `~/.claude/keybindings.json` to customize keyboard shortcuts.

## CRITICAL: Read Before Write

**Always read `~/.claude/keybindings.json` first** (it may not exist yet). Merge changes with existing bindings — never replace the entire file.

- Use **Edit** tool for modifications to existing files
- Use **Write** tool only if the file does not exist yet

## File Format

```json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "$docs": "https://code.claude.com/docs/en/keybindings",
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor"
      }
    }
  ]
}
```

Always include the `$schema` and `$docs` fields.

## Keystroke Syntax

**Modifiers** (combine with `+`):
- `ctrl` (alias: `control`)
- `alt` (aliases: `opt`, `option`) — note: `alt` and `meta` are identical in terminals
- `shift`
- `meta` (aliases: `cmd`, `command`)

**Special keys**: `escape`/`esc`, `enter`/`return`, `tab`, `space`, `backspace`, `delete`, `up`, `down`, `left`, `right`

**Chords**: Space-separated keystrokes, e.g. `ctrl+k ctrl+s` (1-second timeout between keystrokes)

**Examples**: `ctrl+shift+p`, `alt+enter`, `ctrl+k ctrl+n`

## Unbinding Default Shortcuts

Set a key to `null` to remove its default binding:

```json
{
  "context": "Chat",
  "bindings": {
    "ctrl+s": null
  }
}
```

## How User Bindings Interact with Defaults

- User bindings are **additive** — they are appended after the default bindings
- To **move** a binding to a different key: unbind the old key (`null`) AND add the new binding
- A context only needs to appear in the user's file if they want to change something in that context

## Common Patterns

### Rebind a key

To change the external editor shortcut from `ctrl+g` to `ctrl+e`:

```json
{
  "context": "Chat",
  "bindings": {
    "ctrl+g": null,
    "ctrl+e": "chat:externalEditor"
  }
}
```

### Add a chord binding

```json
{
  "context": "Global",
  "bindings": {
    "ctrl+k ctrl+t": "app:toggleTodos"
  }
}
```

## Behavioral Rules

1. Only include contexts the user wants to change (minimal overrides)
2. Validate that actions and contexts are from the known lists
3. Warn the user proactively if they choose a key that conflicts with reserved shortcuts or common tools like tmux (`ctrl+b`) and screen (`ctrl+a`)
4. When adding a new binding for an existing action, the new binding is additive (existing default still works unless explicitly unbound)
5. To fully replace a default binding, unbind the old key AND add the new one

## Available Contexts

| Context | Description |
|---------|-------------|
| `Global` | Always active (lowest priority) |
| `Chat` | Main chat input area |
| `Autocomplete` | Autocomplete menu open |
| `Confirmation` | Yes/No confirmation dialogs |
| `Tabs` | Tab navigation |
| `Transcript` | Message list area |
| `HistorySearch` | History search mode |
| `Task` | Background task view |
| `ThemePicker` | Theme selection menu |
| `Help` | Help dialog |
| `Attachments` | File attachment area |
| `Footer` | Footer status bar |
| `MessageSelector` | Message selection mode |
| `DiffDialog` | Diff viewing dialog |
| `ModelPicker` | Model selection menu |
| `Select` | Selection mode |

## Reserved Shortcuts

### Non-rebindable (errors)

| Key | Reason |
|-----|--------|
| `ctrl+c` | Terminal interrupt (SIGINT) |

### Terminal reserved (errors/warnings)

| Key | Reason |
|-----|--------|
| `ctrl+d` | Terminal EOF |
| `ctrl+z` | Terminal suspend (SIGTSTP) |
| `ctrl+\` | Terminal quit (SIGQUIT) |

### macOS reserved (errors)

| Key | Reason |
|-----|--------|
| `cmd+q` | macOS quit application |
| `cmd+h` | macOS hide application |
| `cmd+m` | macOS minimize window |

## Common Issues and Fixes

| Issue | Cause | Fix |
|-------|-------|-----|
| `keybindings.json must have a "bindings" array` | Missing wrapper object | Wrap bindings in `{ "bindings": [...] }` |
| `"bindings" must be an array` | `bindings` is not an array | Set `"bindings"` to an array |
| `Unknown context "X"` | Typo or invalid context name | Use exact context names from the Available Contexts table |
| `Duplicate key "X" in Y bindings` | Same key defined twice in one context | Remove the duplicate |
| `"X" may not work: ...` | Key conflicts with terminal/OS reserved shortcut | Choose a different key |
| `Could not parse keystroke "X"` | Invalid key syntax | Check syntax: use `+` between modifiers |
| `Invalid action for "X"` | Action value is not a string or null | Actions must be strings like `"app:help"` or `null` |