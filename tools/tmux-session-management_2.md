# tmux Session Management

```bash
# New named session
tmux new -s dev

# Detach: Ctrl-b d
# List sessions
tmux ls
# Attach
tmux a -t dev

# Split panes
Ctrl-b %    # vertical split
Ctrl-b "    # horizontal split

# Navigate panes
Ctrl-b arrow  # or Ctrl-b q then number

# Sync panes (type in all)
Ctrl-b : setw synchronize-panes on

# Save/restore sessions
tmux new -d -s monitor 'htop'
tmux new -d -s logs 'tail -f /var/log/syslog'
```
