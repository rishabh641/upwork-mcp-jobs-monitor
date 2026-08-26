# Setup guide

Step-by-step instructions to run your own copy of the Upwork job monitor. Everything here runs on
*your* Claude subscription, *your* Upwork connector, and *your* Telegram bot — nothing is shared
with anyone else's setup.

Русская версия: [SETUP.ru.md](SETUP.ru.md)

## What you'll need

- A Claude subscription with [Routines](https://claude.ai/code/routines) access.
- An Upwork freelancer account with a complete-ish profile (skills, rate, overview, some
  portfolio/work history). The routine reads this live on every run to judge fit and write
  proposal drafts.
- A GitHub account, to hold your fork.
- A Telegram account, for a bot to message you.

## 1. Fork this repository

Fork it to your own GitHub account. This fork is where the routine will read `profile.md` and
`seen_jobs.json`, and where it commits updated state after each run.

## 2. Connect your Upwork account to Claude

In Claude, open Settings → Connectors and add the **Upwork** connector. Authorize it against your
own Upwork account. This is what lets the routine call `get_profile`, `find_jobs`, and
`get_freelancer_dashboard` on your behalf — it never touches anyone else's account, and it never
submits proposals or applies to jobs; it only reads.

## 3. Create a Telegram bot and get your chat ID

1. Open a chat with [@BotFather](https://t.me/BotFather) on Telegram, send `/newbot`, and follow
   the prompts. You'll get back a **bot token** that looks like `123456789:AAExampleTokenText`.
2. Send any message to your new bot (search for it by the username you gave it and press Start).
3. Get your **chat ID**: open
   `https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates` in a browser (with your real token in
   place of `<YOUR_BOT_TOKEN>`), after sending the bot a message. Look for `"chat":{"id":...}` in
   the response — that number is your chat ID.

Keep both the bot token and chat ID handy for the next step.

## 4. Create the routine

In Claude's Routines UI:

1. Create a new routine and connect it to your fork of this repository.
2. Set the schedule to hourly (this is what the prompt was designed and tested against).
3. Add two secrets/environment variables on the routine:
   - `TELEGRAM_BOT_TOKEN` — the token from step 3.
   - `TELEGRAM_CHAT_ID` — the chat ID from step 3.
4. Make sure the **Upwork MCP connector** is enabled for this routine.
5. Paste the full contents of [`ROUTINE_PROMPT.md`](../ROUTINE_PROMPT.md) as the routine's prompt,
   unmodified.
6. Give the routine write access to your fork (it needs to commit `seen_jobs.json` updates back to
   `main`).
7. Save and enable the routine.

## 5. Customize `profile.md`

Skills, rate, and portfolio are pulled live from your Upwork profile — you don't need to duplicate
them. Edit `profile.md` in your fork only for things Upwork's API can't tell the routine:

- **What to avoid** — e.g. minimum hourly rate, job types you don't want, red flags on clients.
- **Tone** — how you want proposal drafts to sound.
- **Overrides** — only if your live Upwork profile is misleading for scoring purposes (e.g. a
  skill tag you don't actually want work in).

Commit and push this to `main` before the first run.

## 6. What happens on the first run

The very first time the routine runs, `seen_jobs.json` is empty, so it treats that run as a
**seeding run**: it fetches currently-live matching jobs, marks all of their IDs as seen, and
commits — without scoring, drafting, or messaging you. This is intentional, so you don't get
flooded with alerts for every job that already existed before you turned the routine on. Every run
after that alerts only on genuinely new postings.

## Troubleshooting

- **No Telegram messages after the first (seeding) run**: check the bot token and chat ID are set
  correctly as the routine's secrets, and that you've sent the bot at least one message (Telegram
  bots can't message you until you've messaged them first).
- **Too many / irrelevant alerts**: tighten "What to avoid" in `profile.md`.
- **Too few alerts**: loosen "What to avoid", or double check your Upwork profile's skills are
  broad enough to match the jobs you want.
- **Commits aren't showing up in your fork**: make sure the routine has write access to the
  repository.

## Safety notes

This routine only reads your Upwork profile and job matches, reads/writes files in your own
forked repo, and sends Telegram messages. It never submits a proposal, saves a job, or applies to
anything through the connector — you always apply yourself, after reading the alert.
