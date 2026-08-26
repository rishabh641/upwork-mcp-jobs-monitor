# Upwork job monitor — routine template

A [Claude Code routine](https://claude.ai/code/routines) (a scheduled cloud agent) that checks
Upwork for new postings matching your profile every hour, scores each one for fit, drafts a
proposal opening, and sends you a Telegram message. It never submits anything to Upwork or Claude
Cloud — proposals are always sent by you, manually.

This is a **template**: fork it, connect your own Upwork account and Telegram bot, and run your
own copy. Everyone runs it on their own Claude subscription and their own Upwork connector —
there's no shared server or account involved.

**Setup guide:** [docs/SETUP.md](docs/SETUP.md) (English) · [docs/SETUP.ru.md](docs/SETUP.ru.md)
(Русский)

## Files

- `ROUTINE_PROMPT.md` — the exact prompt to paste into your routine. Generic on purpose — no
  names, search terms, or credentials — so you can use it unmodified.
- `profile.md` — preferences Upwork's API has no way of knowing (what to avoid, tone, overrides).
  Skills, rate, and portfolio are pulled live from your Upwork profile via the connector on every
  run, so you don't duplicate them here. Edit this to change what the routine looks for and how
  it writes.
- `seen_jobs.json` — job IDs already alerted on. The routine reads this at the start of each run,
  filters newly-fetched jobs against it, and commits the updated list back to `main` at the end of
  the run so the next run picks up where this one left off. Starts empty on a fresh fork.

There's no app code here on purpose — the routine's logic lives in `ROUTINE_PROMPT.md`, configured
via the Claude Code routines UI/API on your own account. This repo exists so your routine has
somewhere to read and write state between runs, since each run starts from a fresh clone.

## How it works

1. Routine fires hourly → clones your fork fresh from `main`
2. Reads `seen_jobs.json` and `profile.md`
3. Uses your Upwork MCP connector to fetch matching postings
4. Filters out anything already in `seen_jobs.json`
5. For each genuinely new job: scores fit (0-100) and drafts a proposal opening
6. Sends you a Telegram message per new job
7. Appends the new job IDs to `seen_jobs.json` and commits directly to `main`

## Requirements

- A Claude subscription with [Routines](https://claude.ai/code/routines) access.
- An Upwork account (freelancer profile), connected via Claude's Upwork MCP connector.
- A Telegram bot and chat ID to receive alerts.

See the setup guide for step-by-step instructions.
