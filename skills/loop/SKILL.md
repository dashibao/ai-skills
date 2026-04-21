---
name: loop
description: Run a prompt or slash command on a recurring interval (e.g. /loop 5m /foo, defaults to 10m)
---

# /loop — Schedule a Recurring Prompt

Run a prompt or slash command on a recurring interval.

## Usage

```
/loop [interval] <prompt>
```

Intervals: `Ns`, `Nm`, `Nh`, `Nd` (e.g., `5m`, `30m`, `2h`, `1d`). Minimum granularity is 1 minute.
If no interval is specified, defaults to `10m`.

## Examples

```
/loop 5m /babysit-prs
/loop 30m check the deploy
/loop 1h /standup 1
/loop check the deploy          (defaults to 10m)
/loop check the deploy every 20m
```

## Parsing Rules

1. **Leading token**: If the first whitespace-delimited token matches `^\d+[smhd]$` (e.g., `5m`, `2h`), that's the interval; the rest is the prompt.

2. **Trailing "every" clause**: Otherwise, if the input ends with `every <N><unit>` or `every <N> <unit-word>` (e.g., `every 20m`, `every 5 minutes`, `every 2 hours`), extract that as the interval and strip it from the prompt. Only match when what follows "every" is a time expression — `check every PR` has no interval.

3. **Default**: Otherwise, interval is `10m` and the entire input is the prompt.

## Interval → Cron Conversion

| Interval pattern | Cron expression | Notes |
|------------------|-----------------|-------|
| `Nm` where N ≤ 59 | `*/N * * * *` | Every N minutes |
| `Nm` where N ≥ 60 | `0 */H * * *` | Round to hours (H = N/60) |
| `Nh` where N ≤ 23 | `0 */N * * *` | Every N hours |
| `Nd` | `0 0 */N * *` | Every N days at midnight local |
| `Ns` | treat as `ceil(N/60)m` | Cron minimum granularity is 1 minute |

## Important Notes

- Recurring tasks auto-expire after 7 days
- Use `CronDelete` to cancel sooner (job ID shown when created)
- After scheduling, the prompt is immediately executed once
- If it's a slash command, invoke via the Skill tool

## Usage Patterns

| User Request | Command |
|--------------|---------|
| "Check the deploy every 5 minutes" | `/loop 5m check the deploy` |
| "Run /babysit-prs every 30 minutes" | `/loop 30m /babysit-prs` |
| "Check every PR" (no interval) | `/loop check every PR` → interval = 10m |