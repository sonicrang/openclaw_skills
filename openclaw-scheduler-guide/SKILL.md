---
name: openclaw-scheduler-guide
description: Explain when to use OpenClaw heartbeat, OpenClaw cron, or system cron. Use when a user asks how to schedule a task, which scheduler fits a job, or how notifications reach a chat channel.
---

# OpenClaw Scheduler Guide

Use this skill to answer scheduler-choice questions with a short, stable framework.

## Core distinction

- **Heartbeat**: periodic awareness in the **main session**.
- **OpenClaw cron**: OpenClaw **scheduled jobs**.
- **System cron**: OS-level **script/command scheduling**.

## When to use

### Heartbeat

Use when:
- multiple lightweight checks can be batched;
- main-session context matters;
- exact timing does not matter.

Notes:
- runs in the **main session**;
- default delivery is effectively silent unless configured; use heartbeat `target` such as `last` when you want channel delivery;
- if nothing needs attention, `HEARTBEAT_OK` is suppressed.

Example:
- “Every 30 minutes, check inbox + calendar and tell me only if something matters.”

### OpenClaw cron

Use when:
- timing matters;
- the task is an **agent task** with prompt/tool/model behavior;
- the task should run in isolation or deliver directly to a channel.

Notes:
- runs inside the **Gateway** and persists jobs under OpenClaw state;
- supports `main` session jobs and `isolated` jobs;
- isolated jobs default to `announce` delivery if `delivery` is omitted;
- recurring top-of-hour schedules may be staggered by a small deterministic offset unless forced exact.

Example:
- “Every hour, check GitHub issues and notify me if there is something new.”
- “Remind me in 20 minutes.”

Common commands:
- `openclaw cron add ...`
- `openclaw cron list`
- `openclaw cron run <job-id>`
- `openclaw cron remove <job-id>`

### System cron

Use when:
- the task is just “run this script/command on a schedule”; 
- no OpenClaw prompt, session, or tool semantics are needed.

Example:
- “Every hour, run `python monitor.py`.”
- “Every night, back up a directory.”

## Channel delivery

- **Heartbeat**: reply is routed by heartbeat `target` / `to`; `target: "last"` sends to the last contact.
- **OpenClaw cron (main)**: cron enqueues a system event, then main session / heartbeat reply is routed.
- **OpenClaw cron (isolated)**: `delivery.mode = "announce"` sends directly through OpenClaw outbound channel adapters; `channel` and `to` choose the destination.
- **OpenClaw cron (webhook)**: `delivery.mode = "webhook"` posts to a URL, not to a chat channel.
- **System cron**: no native OpenClaw delivery; the script must call an API or invoke OpenClaw.

## Default answer pattern

Answer in this order:
1. one-line distinction;
2. which one fits this task;
3. one short reason;
4. one short example.
