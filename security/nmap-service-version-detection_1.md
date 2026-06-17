# Nmap Service Version Detection

Using `-sV` to enumerate service versions for vulnerability matching.

```bash
# Standard version detection
nmap -sV 10.10.10.10

# Intensity 0-9 (higher = more probes)
nmap -sV --version-intensity 5 10.10.10.10

# All probes (slow but thorough)
nmap -sV --version-all 10.10.10.10

# Light version detection (fast)
nmap -sV --version-light 10.10.10.10
```

Output includes service name, version, and sometimes OS info for CVE matching.
