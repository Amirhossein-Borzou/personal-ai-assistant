---
name: stock-watchdog
description: "Use when monitoring a stock's intraday drop: threshold alerts with multi-channel escalation and rate limiting."
version: 1.2.0
---

# Stock Watchdog

Watches one symbol and alerts on abnormal intraday losses. Designed for a **script-only cron job** (no LLM) that escalates to the agent only when a threshold trips.

## Alert rules

| Condition | Action |
|---|---|
| Day drop ≤ -3.5% | Alert immediately |
| Sudden move ≤ -1.5% in 30 min (total ≤ -2.5%) | Alert immediately |
| Otherwise | Silent |

## Data sources (fallback chain)

1. TradingView — primary quote endpoint
2. Nasdaq — fallback when primary rate-limits (Yahoo returns 429 on many VPS ranges)

## Escalation chain

1. **Script cron (no LLM)** — cheap check every 30 min; silent unless a threshold trips
2. **Agent run** — triggered when the script prints output; adds context and headlines
3. **Delivery** — chat message; retries until acknowledged

## Anti-flapping

- One alert per rule per session; reset only on recovery
- Cooldown of 60 min between repeat alerts for the same rule

## Example agent prompt (for the LLM escalation step)

```
NVDA watchdog tripped: {{day_change}}% today, {{last_30m}}% in 30 min.
Fetch current quote, add 2-3 relevant headlines in Persian, and deliver
a single alert message. Do not repeat the raw numbers already shown.
```
