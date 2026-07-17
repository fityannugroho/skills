# `tee` vs `capture-pane`

Both can capture output, but work differently.

## Comparison

| | `tee` | `capture-pane` |
|---|---|---|
| **When to capture** | Real-time — while command runs | Snapshot — when requested |
| **What is captured** | Output piped to tee | Visual terminal content (including scrollback) |
| **Scrollback** | ❌ Not included | ✅ Available with `-S -1000` |
| **When to use** | Save all output for later review | Check what's visible in tmux right now |

## Examples

```bash
# tee — record all output to file (streaming recorder)
tmux new-session -d -s bg-build "make 2>&1 | tee build.log"

# capture-pane — take terminal screenshot now
tmux capture-pane -t bg-build -p
```

## When to use which

- **Save complete log** → use `tee`
- **Quick progress check** → use `capture-pane`
- **Debug what's happening** → use `capture-pane`

## Interactive output (progress bars, spinners, ncurses)

Commands with interactive output (progress bars, spinners, ncurses UIs, htop-style) **overwrite the same terminal lines** using escape codes. `tee` logs become useless — you get garbled ANSI escape sequences instead of readable progress.

**Always use `capture-pane`** for these commands. It snapshots the actual rendered screen, not the raw stream.

```bash
# BAD — tee captures escape codes, not the progress bar
tmux new-session -d -s bg-dl "wget url 2>&1 | tee dl.log"

# GOOD — capture-pane gets the actual visual state
tmux new-session -d -s bg-dl "wget url"
tmux capture-pane -t bg-dl -p
```
