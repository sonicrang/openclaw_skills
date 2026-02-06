---
name: open-ralph-wiggum
description: Run Open Ralph Wiggum AI coding agent in tmux mode.
metadata:
  openclaw:
    emoji: "🤖"
    os: ["darwin", "linux"]
    requires:
      bins: ["ralph", "tmux"]
---

# Open Ralph Wiggum

Run ralph AI coding agent in headless tmux sessions.

## ⚠️ CRITICAL: MUST ASK USER BEFORE STARTING

**DO NOT start ralph without confirming these two required parameters:**

### 1. Project Path (Required)
- Where should ralph create/work on the project?
- Example: `/root/projects/my-app` or `./my-project`

### 2. Max Iterations (Required)
- How many iterations should ralph run?
- Suggest 10-30 depending on task complexity
- Example: `--max-iterations 20`

**❌ WRONG:** Starting ralph with a default path or without asking
**✅ CORRECT:** Ask user, get confirmation, then start

---

## Prerequisites

```bash
npm install -g @th0rgal/ralph-wiggum
```

Requires: `tmux`

## Usage Flow

### Step 1: Confirm with User (REQUIRED)

**MUST ask the user before starting:**

1. **Project path** - Where to create the project?
2. **Max iterations** - How many? (suggest 10-30)

Wait for user response. Do not proceed without these.

### Step 2: Start Ralph

Use `ralph` directly with tmux:

```bash
# Create tmux session
tmux -S /tmp/ralph-$(whoami).sock new-session -d -s ralph -c /path/to/project

# Run ralph
tmux -S /tmp/ralph-$(whoami).sock send-keys -t ralph:0.0 -l 'ralph "<task>" --no-stream --allow-all --agent opencode --max-iterations <N>'
tmux -S /tmp/ralph-$(whoami).sock send-keys -t ralph:0.0 Enter
```

Example:
```bash
ralph "Build a REST API" --no-stream --allow-all --agent opencode --max-iterations 20
```

## Session Details

| Item | Path |
|------|------|
| Socket | `/tmp/ralph-$(whoami).sock` |
| Session | `ralph` |

## Complete Example

```bash
cd /path/to/project

# Start ralph directly (after confirming path and iterations with user)
ralph "Create API" --no-stream --allow-all --agent opencode --max-iterations 10

# Watch live in tmux
tmux -S /tmp/ralph-$(whoami).sock attach -t ralph
# Detach: Ctrl+B, then D
```
