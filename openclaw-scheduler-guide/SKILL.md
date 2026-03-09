---
name: openclaw-scheduler-guide
description: Explain when to use OpenClaw heartbeat, OpenClaw cron, or system cron. Use when a user asks how to schedule a task, which scheduler fits a job, how notifications reach a chat channel, or how to configure reliable scheduled notifications. Prefer this skill especially when the user wants scheduled tasks to actively notify them.
---

# OpenClaw Scheduler Guide

Use this skill to answer scheduler-choice questions with a short, stable framework.

## Core distinction

- **Heartbeat**: periodic awareness in the **main session**.
- **OpenClaw cron**: OpenClaw **scheduled jobs**.
- **System cron**: OS-level **script/command scheduling**.

## Hard rule for notification tasks

If the user wants a scheduled task to **actively notify them in chat**, recommend:

- **OpenClaw cron**
- **`--session isolated`**
- explicit delivery settings: `--announce --channel <channel> --to <destination>`

Do **not** default to these for notification-critical tasks unless the user explicitly asks for them and accepts the tradeoff:

- **Heartbeat**
- **OpenClaw cron `main` session**
- **System cron**

Reason:
- heartbeat may be silent by design and depends on heartbeat routing/config;
- cron `main` depends on main-session delivery behavior instead of direct delivery;
- system cron has no native OpenClaw chat delivery.

So for requests like:
- “定时提醒我”
- “跑完后通知我”
- “定时检查并发消息给我”
- “set up a scheduled job and notify me”

Default to:
- **OpenClaw cron in `isolated` mode with explicit `--announce --channel --to`.**

## Notification-safe OpenClaw cron pattern

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

Key flags:
- `--session isolated`: run outside the main session; avoid heartbeat/main-session delivery ambiguity.
- `--announce`: enable direct outbound delivery for the cron result.
- `--channel <channel>`: choose the destination channel explicitly.
- `--to <destination>`: choose the exact recipient/chat/channel explicitly.

Destination rule:
- do **not** hardcode personal identifiers in the skill;
- if the job should notify the **current chat**, auto-fill `--channel` and `--to` from the current session/chat context;
- if the job should notify **another destination**, ask the user to confirm the target or resolve the target id first.

Useful optional flags:
- `--light-context`: reduce unnecessary context for routine scheduled jobs.
- `--timeout-seconds <n>`: give the run enough time.
- `--exact`: disable cron staggering when exact timing matters.

Model rule:
- do **not** override the model by default;
- only add `--model <model>` when the user explicitly asks for a specific model.

## After configuration: verify persisted cron config, then ask for a test

When you create, edit, migrate, or retarget an OpenClaw cron job, do **not** stop after the CLI reports success. Also verify the persisted scheduler file:

```text
~/.openclaw/cron/jobs.json
```

Why this matters:
- the job's actual persisted payload/path/delivery config lives in `jobs.json`;
- when moving scripts or changing execution paths, a stale `payload.message` in `jobs.json` can leave the cron job pointing at an old path even if the user thinks the task was "updated";
- after any cron edit related to script path, message, schedule, or delivery target, check `jobs.json` and confirm the expected fields were updated.

Minimum verification checklist after cron changes:
1. confirm the target job id in `openclaw cron list`;
2. inspect `~/.openclaw/cron/jobs.json` for the updated `schedule` and `payload.message`;
3. only then treat the edit as complete;
4. then ask for a manual test.

After setting up a scheduled notification job, explicitly guide the user to validate it.

Preferred follow-up:
1. run the job once manually with `openclaw cron run <job-id>`;
2. verify that the notification reached the intended chat;
3. verify that the content format matches expectation;
4. only then treat the setup as complete.

Use wording like:
- “Let’s test it once now.”
- “Run `openclaw cron run <job-id>` and confirm you received the message.”
- “If nothing arrives, check delivery settings before relying on the schedule.”

## Default answer pattern

Answer in this order:
1. give the recommendation first: for scheduled notifications, use **OpenClaw cron isolated**;
2. if needed, briefly explain why heartbeat / cron main / system cron are not the default for this case;
3. show the concrete command pattern with `--session isolated --announce --channel --to`;
4. do not add `--model` unless the user explicitly requested it;
5. end by asking the user to run a manual test and confirm delivery.
