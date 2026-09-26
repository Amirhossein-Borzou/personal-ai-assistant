# Example cron jobs

These are real patterns this assistant runs. Created by typing plain text in chat — no crontab editing.

## LLM jobs (agent wakes up and reasons)

| Job | Prompt pattern |
|---|---|
| Morning briefing | *"Every day at 07:25, send today's calendar events and weather."* |
| Weekly digest | *"Mondays at 09:00, summarize last week's completed tasks and stalled ones."* |
| Price/news watcher | *"Every hour, check the NVDA price; alert me only if it drops 3.5%+ today, plus relevant Persian headlines."* |

## Script-only jobs (no LLM — cheap watchdogs)

A script runs on schedule; its stdout is delivered verbatim and **the LLM is never woken** unless output changes:

```bash
# watchdog.sh — exits silently unless threshold trips
#!/bin/bash
# threshold logic here; print alert text ONLY when it trips
```

Best for: disk alerts, health checks, price thresholds, process liveness.

## Design rules that make these reliable

1. **Idempotent prompts** — the agent may be a fresh session each run; the prompt must be self-contained.
2. **Silence by default** — watchers only speak when something changed.
3. **Escalation chain** — try the cheap channel first (script), escalate to LLM analysis, then to multiple delivery channels.
