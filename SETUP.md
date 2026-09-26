# Demo Environment Setup

Step-by-step guide to reproduce this assistant in a **safe, isolated demo profile** — so you can demo it publicly without exposing any real data.

## 0. Why an isolated profile?

A Hermes *profile* is a fully isolated home directory: its own `config.yaml`, `.env`, memories, sessions, skills, and cron jobs. The demo profile shares nothing with your production profile — not the bot token, not the mailbox, not the calendar.

## 1. Install Hermes Agent

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
hermes doctor   # health check
```

## 2. Create the demo profile

```bash
hermes profile create demo
demo setup      # configure model + API keys for this profile only
```

This creates a new `demo` command. Everything you run through `demo ...` stays inside `~/.hermes/profiles/demo/`.

## 3. Create a throwaway Telegram bot

1. Message [@BotFather](https://t.me/BotFather) → `/newbot` → name it e.g. `DemoAssistant`
2. Copy the token into the **demo profile only**:

```bash
demo config set gateway.telegram.enabled true
# paste the token when prompted by setup, or into the profile's .env
```

Never reuse your production bot token. The demo bot has zero real chat history.

## 4. Connect Google (Calendar + Gmail)

1. [Google Cloud Console](https://console.cloud.google.com) → new project → enable **Calendar API** and **Gmail API**
2. OAuth consent screen → *Testing* → add only your own throwaway Google account
3. Create OAuth client (Desktop app) → download credentials into the demo profile
4. Run the OAuth flow inside the demo profile and grant access to the throwaway account only

> For the public demo, seed the calendar with synthetic events ("Team sync — 10:00") and the mailbox with fake emails. **Keep Gmail read-only**: the assistant drafts, a human sends.

## 5. Add the example skill + cron jobs

```bash
# custom skill (threshold watchdog)
mkdir -p ~/.hermes/profiles/demo/skills
cp -r skills/stock-watchdog ~/.hermes/profiles/demo/skills/

# scheduled jobs — create from plain text in chat, e.g.:
#   "Every weekday at 07:25, summarize my calendar for today"
#   "Check NVDA every 30 minutes; alert me if it drops more than 3.5% today"
```

See `cron-examples/` for the job patterns.

## 6. Record the demos

Record short screen captures of the **demo bot** and convert to GIF:

```bash
# example with ffmpeg (after recording screenshots or a screen capture)
ffmpeg -i demo.mp4 -vf "fps=12,scale=720:-1:flags=lanczos" -loop 0 demo.gif
```

Suggested shots (all with synthetic data):

| File | Show this |
|---|---|
| `demos/calendar-briefing.gif` | "What's on my calendar today?" → real answer from seeded events |
| `demos/email-triage.gif` | "Summarize my unread emails" → triage of fake inbox |
| `demos/cron-setup.gif` | Typing a plain-text schedule → job created → alert arrives on time |
| `demos/multi-agent.gif` | Delegating a task to a spawned agent, result returns to chat |
| `demos/web-research.gif` | Asking for sources → cited answer |

Checklist before exporting anything:
- [ ] Demo bot only — production bot nowhere on screen
- [ ] No real names, emails, phone numbers, calendar entries
- [ ] No terminal output containing paths, tokens, or real config
- [ ] GIFs re-watched frame by frame before commit

## 7. Run it on boot (VPS)

```bash
demo gateway start       # start the demo bot's gateway
# persist with systemd per the Hermes docs: https://hermes-agent.nousresearch.com/docs/
```
