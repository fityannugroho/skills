# Log Management

## How Logging Works

Each background task produces 2 log files:

| File | Content |
|------|---------|
| `<log-dir>/<label>.log` | stdout + stderr from the command |
| `<log-dir>/<label>.exit` | exit code (0 = success) |

Logs are captured using `tee` which writes to a file AND displays to the tmux terminal simultaneously.

## Log Directory Options

Logs can be saved anywhere:

| Location | Example | When to use |
|----------|---------|-------------|
| **Project-local** | `./logs/`, `./tmp/` | Project-specific logs, can be gitignored |
| **User cache** | `~/.cache/bg-logs/` | Temporary logs, not important |
| **User data** | `~/.local/share/bg-logs/` | Persistent logs, important |
| **Custom** | `/tmp/bg-logs/`, `~/logs/` | Anywhere |

**Default:** `~/.opencode/bg-logs/`

## Start Task with Logging

```bash
LOG_DIR="<where to save logs>"

tmux new-session -d -s bg-build \
    "cd /project && make 2>&1 | tee $LOG_DIR/build.log; \
    echo \$? > $LOG_DIR/build.exit"
```

**Examples by location:**

```bash
# Project-local
tmux new-session -d -s bg-build \
    "make 2>&1 | tee ./logs/build.log; echo \$? > ./logs/build.exit"

# User cache
tmux new-session -d -s bg-build \
    "make 2>&1 | tee ~/.cache/bg-logs/build.log; echo \$? > ~/.cache/bg-logs/build.exit"

# Temp directory
tmux new-session -d -s bg-build \
    "make 2>&1 | tee /tmp/bg-build.log; echo \$? > /tmp/bg-build.exit"
```

## Read Logs

```bash
# View all output
cat <log-dir>/build.log

# View last 50 lines
tail -n 50 <log-dir>/build.log

# View exit code
cat <log-dir>/build.exit

# Search for pattern in log
grep "error" <log-dir>/build.log
```

## List All Logs

```bash
# All log files in a location
ls <log-dir>/

# Only existing logs
ls <log-dir>/*.log 2>/dev/null
```

## Cleanup Logs

```bash
# Remove specific task logs
rm <log-dir>/build.log <log-dir>/build.exit

# Remove all bg task logs
rm <log-dir>/*.log <log-dir>/*.exit 2>/dev/null

# Remove old logs (older than 7 days)
find <log-dir> -name "*.log" -mtime +7 -delete
find <log-dir> -name "*.exit" -mtime +7 -delete
```
