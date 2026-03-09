---
name: openclaw-scheduler-guide
description: Configure or explain scheduled tasks that need chat notifications in OpenClaw. Use when the user asks to create, edit, migrate, or choose a scheduler for any 定时+通知 task, reminder, periodic check, or "run and notify me" workflow. Default to OpenClaw cron with `--session isolated`; do not choose Heartbeat, OpenClaw cron main session, or system cron unless the user explicitly asks for them.
---

# OpenClaw Scheduler Guide

## Default rule

For any task that is both **scheduled** and **needs notification**, default to:

- **OpenClaw cron**
- **`--session isolated`**
- **explicit delivery**: `--announce --channel <channel> --to <destination>`

Apply this default to requests like:
- “定时提醒我”
- “每天跑一次并通知我”
- “每小时检查 XXX，有结果发给我”
- “set up a scheduled job and notify me”

Do **not** default to these unless the user explicitly asks for them:
- **Heartbeat**
- **OpenClaw cron `main` session**
- **System cron**

## Command pattern

Start from this shape:

```bash
openclaw cron add \
  --name "Job name" \
  --cron "0 * * * *" \
  --session isolated \
  --message "Do the task. If there is nothing worth reporting, stay silent." \
  --announce \
  --channel <channel> \
  --to <destination>
```

Use these flags deliberately:
- `--session isolated`: run in an isolated cron session and avoid main-session delivery ambiguity.
- `--announce`: send the result outward instead of keeping it only in session history.
- `--channel <channel>`: set the destination channel explicitly.
- `--to <destination>`: set the exact chat/user/channel destination explicitly.

Useful optional flags:
- `--light-context`: prefer for routine jobs.
- `--timeout-seconds <n>`: increase when the task may take longer.
- `--exact`: use when precise timing matters.

Do **not** add `--model` unless the user explicitly asks for a specific model.

## How to get `channel` and `to`

Destination rule:
- If the user wants the result sent to the **current chat**, fill `--channel` and `--to` from the current session/chat metadata.
- If the user wants another destination, ask for confirmation or resolve the target first.
- Do **not** hardcode personal identifiers into this skill.

## Testing rule

For periodic scheduled jobs, require a test after setup or edit.

Minimum validation flow:
1. confirm the job exists in `openclaw cron list`;
2. verify the persisted config in `~/.openclaw/cron/jobs.json`, especially `schedule`, `payload.message`, and delivery fields;
3. run `openclaw cron run <job-id>` once;
4. confirm the notification actually arrived in the intended chat.

Default closing move:
- ask the user to test once now before relying on the schedule.
