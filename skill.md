---
name: reminder-guardian
description: Log every reminder, keep Cron blueprints aligned, and always source the current time through the shared helper before mentioning it.
---

# Reminder Guardian

This skill keeps reminder scheduling from becoming a half-finished promise. It pairs a lightweight CLI with its bundled `scripts/time_helper.py` so you always (1) log a reminder, (2) copy a reproducible Cron blueprint, and (3) verify the reminder eventually fires.

## What it contains

- **`scripts/reminder_guard.py`**: CLI to create reminder entries, mark them scheduled/sent, and print the `openclaw cron add` blueprint you need to run manually. Each entry stores `id`, `message`, `time`, `status`, and optional notes inside `memory/reminder-log.json` so you can audit skipped reminders later.
- **Time helper integration**: The skill ships with `scripts/time_helper.py`, and every command that needs the current timestamp calls it so the reminder workflow always uses the canonical clock. The CLI never relies on an external copy of the helper.

## Workflow

1. **Add a reminder**
   ```bash
   python3 skills/reminder-guardian/scripts/reminder_guard.py \
     add --message "Daily mood check-in" --offset +30m --label "Mood" --note "Before evening call"
   ```
   - The CLI prints a Cron blueprint (schedule + payload) and logs the entry with status `pending`.

2. **Schedule it with `cron`**
   Copy the blueprint and run a matching `openclaw cron add` command (choose `announce`, `signal`, etc., whichever channel you want). After the job is created, mark the reminder as scheduled:
   ```bash
   python3 skills/reminder-guardian/scripts/reminder_guard.py blueprint 3 --mark
   ```
   That records the change so nothing slips through the cracks.

3. **Update when the reminder fires**
   After you deliver the reminder (Cron job runs), mark it as sent:
   ```bash
   python3 skills/reminder-guardian/scripts/reminder_guard.py update 3 --status sent
   ```

4. **Check the queue**
   ```bash
   python3 skills/reminder-guardian/scripts/reminder_guard.py next
   ```
   You’ll see the next pending reminder and its blueprint to re-run if needed.

## Always use the time helper

Whenever you mention *what time* a reminder should go out or log a dose, run `python3 scripts/time_helper.py` first and quote the ISO string it returns. This skill enforces that rule by calling `scripts/time_helper.py` internally, so the CLI never invents its own clock.

## Publishing

This folder is a standalone AgentSkill. Commit `skills/reminder-guardian/` plus the `.gitignore` change for `memory/reminder-log.json`, push to your GitHub repo, and the skill is ready for others to install. When you publish, highlight the workflow above so any future assistant knows to run the CLI before creating or mentioning reminders.
