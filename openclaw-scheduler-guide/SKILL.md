---
name: openclaw-scheduler-guide
description: Configure or explain scheduled tasks and reliable notification delivery in OpenClaw. Use when the user asks to create, edit, migrate, or choose a scheduler for any 定时+通知 task, reminder, periodic check, or "run and notify me" workflow, or when you need to ensure a background task, subagent completion, cron run, or proactive send actually reaches the user. Default to OpenClaw cron with `--session isolated`; do not choose Heartbeat, OpenClaw cron main session, or system cron unless the user explicitly asks for them.
---

# OpenClaw Scheduler Guide

## Default rule

If you promise a future user-visible update — for example, “稍后发你”, “完成后通知你”, “我发给你”, “跑完再告诉你” — you must also define the actual delivery path.

That means you need to know:
1. what event counts as completion;
2. who sends the later message;
3. which OpenClaw path/tool delivers it.

Do not rely on vague intent or assume a background task will somehow create a later visible message automatically.

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

## Async delivery rules

### 1. Short task: reply when finished

If the task is likely to finish in the current turn, do not say “I’ll send it later”.
Just finish the work and reply once with the result.

### 2. Scheduled or periodic task: use cron with explicit delivery

For anything like “every 10 minutes tell me status”, “run later and notify me”, or recurring checks, use OpenClaw cron with explicit delivery:

- `--announce`
- `--channel <channel>`
- `--to <destination>`

Starting a cron job without delivery fields is not enough if the user expects a visible notification.

### 3. Background exec/process: process start is not delivery

Starting a background `exec` or managing a `process` only means the work is running.
It does **not** mean completion will be announced.
If the user expects a later message, add a concrete follow-up path:
- cron watcher with delivery, or
- stay with the task and send the completion update yourself later.

### 4. Subagent completion: spawn is not final delivery

`sessions_spawn` is non-blocking. Spawning a subagent does not mean the final result has already been sent.

For subagent work:
- If you need the final result in chat, either wait for the completion event and send the update yourself, or explicitly use a separate proactive send path.
- Do not say “I’ll send you the result later” unless you actually control that later send.
- If a completion event arrives and asks for user delivery, rewrite it in your own voice and send it immediately.

### 5. `message` + `NO_REPLY`: when to use it

Use `message(action=send)` when you want to proactively deliver the user-visible message through the messaging tool.

After a successful `message(action=send)` that serves as the visible reply, return only:

`NO_REPLY`

Why:
- it prevents duplicate delivery;
- the user-visible content has already been sent by the tool.

Do **not** use a normal assistant reply after a successful proactive `message(action=send)` for the same content.

Do **not** confuse this with future follow-up delivery:
- `NO_REPLY` only suppresses the current turn’s extra reply;
- it does not create a later completion message by itself.

## How to get `channel` and `to`

Destination rule:
- If the user wants the result sent to the **current chat**, fill `--channel` and `--to` from the current session/chat metadata.
- If the user wants another destination, ask for confirmation or resolve the target first.
- Do **not** hardcode personal identifiers into this skill.

## Common safe phrasing

Prefer exact status language.

Good:
- “我先在后台运行；现在还没最终结果。”
- “我已经设置好每 10 分钟一次通知。”
- “这条是开始确认，完成结果我会另外发一条。”
- “我先把已完成的这一份发你，剩下的还没发。”

Avoid:
- “我稍后发你” with no delivery path
- “完成后通知你” when you have not set up who/how sends it
- “已经在跟进了” when nothing user-visible will arrive

## Testing rule

For periodic scheduled jobs, require a test after setup or edit.

Minimum validation flow:
1. confirm the job exists in `openclaw cron list`;
2. verify the persisted config in `~/.openclaw/cron/jobs.json`, especially `schedule`, `payload.message`, and delivery fields;
3. run `openclaw cron run <job-id>` once;
4. confirm the notification actually arrived in the intended chat.

Default closing move:
- ask the user to test once now before relying on the schedule.
