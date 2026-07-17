---
name: background-exec
description: Run long-running commands in background using tmux. TRIGGER on: any command taking >5s, builds, compilations, migrations, deploys, tests. TRIGGER on phrases: background, run in background, don't block, don't wait, run later. ALWAYS use for heavy commands instead of blocking.
author: fityannugroho
license: MIT
---

# Background Execution with tmux

Run commands in background using tmux. The agent starts a task, checks status, retrieves output — all without blocking.

## Quick Start

```bash
# 1. Set log directory
LOG_DIR="./logs"
mkdir -p "$LOG_DIR"

# 2. Start task
tmux new-session -d -s bg-build \
    "make -j4 2>&1 | tee $LOG_DIR/build.log; echo \$? > $LOG_DIR/build.exit"

# 3. Check status
tmux has-session -t bg-build 2>/dev/null && echo "running" || echo "done"

# 4. Get output
cat $LOG_DIR/build.log
```

## Essential Patterns

### 1. Start a Background Task

```bash
tmux new-session -d -s <session-name> \
    "<command> 2>&1 | tee $LOG_DIR/<label>.log; \
    echo \$? > $LOG_DIR/<label>.exit"
```

**Key flags:**
- `-d` — detached (runs in background)
- `-s` — session name
- `-c` — working directory (optional)

### 2. Check Status

```bash
# Returns 0 if running, 1 if not
tmux has-session -t <session-name> 2>/dev/null
```

### 3. Get Output

```bash
# From log file
cat $LOG_DIR/<label>.log

# Last N lines
tail -n 50 $LOG_DIR/<label>.log

# Live follow
tail -f $LOG_DIR/<label>.log

# Exit code
cat $LOG_DIR/<label>.exit
```

### 4. Kill a Task

```bash
# Graceful (Ctrl-C)
tmux send-keys -t <session-name> C-c

# Force kill
tmux kill-session -t <session-name>
```

### 5. List Tasks

```bash
# All sessions
tmux ls

# Only bg sessions
tmux ls -F "#{session_name}" | grep "^bg-"
```

### 6. Send Keystrokes

```bash
# Answer a prompt
tmux send-keys -t <session-name> "y" Enter

# Type a command
tmux send-keys -t <session-name> "ls -la" Enter
```

### 7. Capture Pane

```bash
# Current visible content
tmux capture-pane -t <session-name> -p

# With scrollback (last 1000 lines)
tmux capture-pane -t <session-name> -p -S -1000
```

> See `references/tee-vs-capture.md` for when to use `tee` vs `capture-pane`.

### 8. Attach to Session

```bash
tmux attach -t <session-name>
# Detach: Ctrl-b then d
```

## Command with Variables

**Use single quotes** to prevent shell expansion:

```bash
tmux new-session -d -s bg-task 'for i in 1 2 3; do echo "Line $i"; done'
```

**Or escape variables:**

```bash
tmux new-session -d -s bg-task "for i in 1 2 3; do echo \"Line \$i\"; done"
```

## Common Patterns

### Wait for completion

```bash
while tmux has-session -t bg-task 2>/dev/null; do sleep 2; done
```

### Run multiple tasks in parallel

```bash
tmux new-session -d -s bg-task-a "command-a 2>&1 | tee $LOG_DIR/a.log; echo \$? > $LOG_DIR/a.exit"
tmux new-session -d -s bg-task-b "command-b 2>&1 | tee $LOG_DIR/b.log; echo \$? > $LOG_DIR/b.exit"

while tmux has-session -t bg-task-a 2>/dev/null || tmux has-session -t bg-task-b 2>/dev/null; do
    sleep 2
done
```

### Timeout

```bash
tmux new-session -d -s bg-task "timeout 300 long-command 2>&1 | tee $LOG_DIR/task.log; echo \$? > $LOG_DIR/task.exit"
```

### Retry loop

```bash
tmux new-session -d -s bg-task "while true; do command && break; sleep 5; done 2>&1 | tee $LOG_DIR/task.log; echo \$? > $LOG_DIR/task.exit"
```

## Session Naming

Use `bg-` prefix for all background task sessions:

```
bg-build
bg-docker
bg-migration
```

## Cleanup

```bash
# Remove specific logs
rm $LOG_DIR/<label>.log $LOG_DIR/<label>.exit

# Kill all bg sessions
for s in $(tmux ls -F "#{session_name}" 2>/dev/null | grep "^bg-"); do
    tmux kill-session -t "$s"
done
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `tmux: command not found` | Install: `apt install tmux` or `brew install tmux` |
| Session not found | Check `tmux ls` — session may have exited already |
| Output is empty | Command may have redirected output; check the command |
| Variable not expanding | Use single quotes or escape with `\$` |

## References

- `references/log-management.md` — Detailed log management guide
- `references/tee-vs-capture.md` — Comparison of tee vs capture-pane
