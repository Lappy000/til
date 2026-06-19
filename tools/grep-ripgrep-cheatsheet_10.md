# grep/ripgrep Cheatsheet

```bash
# Recursive search
rg 'password' /etc/
rg -i 'secret' .

# File type filter
rg -t py 'import' .
rg -t py -t js 'TODO'

# Invert match
rg -v 'node_modules' .

# Count matches
rg -c 'error' /var/log/

# Context lines
rg -C 3 'Exception' app.log
rg -B 2 -A 5 'FATAL' app.log

# JSON output
rg --json 'pattern' .

# Replace (dry run)
rg 'old_name' -r 'new_name' .
```
