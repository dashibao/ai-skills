---
name: schedule
description: Create, update, list, or run scheduled remote agents (triggers) that execute on a cron schedule.
---

# Schedule Remote Agents

You are helping the user schedule, update, list, or run **remote** Claude Code agents. These are NOT local cron jobs — each trigger spawns a fully isolated remote session in Anthropic's cloud infrastructure on a cron schedule.

## What You Can Do

- **Create** — Schedule a new remote agent with a cron expression
- **List** — Show all existing triggers
- **Update** — Modify an existing trigger's schedule, prompt, or settings
- **Run now** — Execute a trigger immediately (outside its schedule)

## First Step

Ask the user what they want to do: create, list, update, or run a trigger.

## Workflow

### CREATE a new trigger

1. **Understand the goal** — Ask what they want the remote agent to do. What repo(s)? What task? Remind them that the agent runs remotely — it won't have access to their local machine.

2. **Craft the prompt** — Help them write an effective agent prompt. Good prompts are:
   - Specific about what to do and what success looks like
   - Clear about which files/areas to focus on
   - Explicit about what actions to take (open PRs, commit, just analyze, etc.)

3. **Set the schedule** — Ask when and how often. Minimum interval is 1 hour.

4. **Choose the model** — Default to `claude-sonnet-4-6`. Ask if they want a different one.

5. **Review and confirm** — Show the full configuration before creating.

6. **Create it** — Show the result with the trigger ID and link to `https://claude.ai/code/scheduled/{TRIGGER_ID}`

### UPDATE a trigger

1. List triggers first so they can pick one
2. Ask what they want to change
3. Show current vs proposed value
4. Confirm and update

### LIST triggers

1. Fetch and display in a readable format
2. Show: name, schedule (human-readable), enabled/disabled, next run, repo(s)

### RUN NOW

1. List triggers if they haven't specified which one
2. Confirm which trigger
3. Execute and confirm

## Cron Expression Examples

- `0 9 * * 1-5` — Every weekday at 9am UTC
- `0 */2 * * *` — Every 2 hours
- `0 0 * * *` — Daily at midnight UTC
- `30 14 * * 1` — Every Monday at 2:30pm UTC
- `0 8 1 * *` — First of every month at 8am UTC

Minimum interval is 1 hour. `*/30 * * * *` will be rejected.

## Important Notes

- These are REMOTE agents — they run in Anthropic's cloud, not on the user's machine
- They cannot access local files, local services, or local environment variables
- Always convert cron to human-readable when displaying
- Default to `enabled: true` unless user says otherwise
- To delete a trigger, direct users to https://claude.ai/code/scheduled

## Requirements

- Must have a claude.ai account authenticated
- API accounts are not supported — run `/login` first

## Usage

```
/schedule                     # Interactive: choose create/list/update/run
/schedule create daily PR check at 9am
/schedule list
/schedule run <trigger-id>
```