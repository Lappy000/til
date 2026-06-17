# Nmap NSE Scripts for Vulnerability Scanning

Using Nmap Scripting Engine (NSE) for automated vulnerability detection.

## Categories

```bash
# List all categories
nmap --script-help all | grep "Categories:"

# Main categories:
# auth        - authentication bypass
# brute       - brute force
# default     - safe default scripts
# exploit     - known exploits
# fuzzer      - fuzzing
# intrusive   - may crash services
# malware     - malware detection
# safe        - non-intrusive
# vuln        - vulnerability detection
```

## Quick Vulnerability Scan

```bash
# Run all vuln scripts against a target
nmap -sV --script vuln 10.10.10.10

# Specific vulnerability checks
nmap --script smb-vuln* 10.10.10.10           # SMB vulns
nmap --script ssl-* 10.10.10.10               # SSL/TLS issues
nmap --script http-vuln* 10.10.10.10          # HTTP vulnerabilities
nmap --script dns-* 10.10.10.10               # DNS issues
```

## Useful Individual Scripts

```bash
# EternalBlue (MS17-010)
nmap -p445 --script smb-vuln-ms17-010 10.10.10.10

# Shellshock
nmap -p80,443 --script http-shellshock --script-args uri=/cgi-bin/test 10.10.10.10

# Heartbleed
nmap -p443 --script ssl-heartbleed 10.10.10.10

# SQL injection detection
nmap -p80 --script http-sql-injection 10.10.10.10

# Default credentials
nmap --script smb-enum-shares,smb-enum-users -p445 10.10.10.10

# HTTP enum
nmap -p80,443 --script http-enum,http-headers,http-methods,http-title 10.10.10.10

# FTP anonymous login
nmap -p21 --script ftp-anon,ftp-syst 10.10.10.10
```

## Script Arguments

```bash
# Pass arguments
nmap --script http-enum --script-args http-enum.basepath=/admin/ 10.10.10.10

# Multiple scripts with shared args
nmap --script smb-vuln*,smb-enum*   --script-args smbuser=admin,smbpass=password   -p445 10.10.10.10
```

## Output for Reporting

```bash
# XML output (parseable) + grepable
nmap -sV --script vuln -oX scan.xml -oG scan.gnmap 10.10.10.10

# Parse with python
python3 -c "
import xml.etree.ElementTree as ET
tree = ET.parse('scan.xml')
for host in tree.findall('.//host'):
    addr = host.find('address').get('addr')
    for script in host.findall('.//script'):
        print(f'{addr} | {script.get("id")}: {script.get("output")[:200]}')
"
```

## Speed Tips

```bash
# Fast scan: top ports + vuln scripts
nmap -F --script vuln --max-parallelism 100 -T4 10.10.10.10

# Scan range with timing
nmap -sV --script default,safe -T4 --min-rate 1000 10.10.10.0/24
```
