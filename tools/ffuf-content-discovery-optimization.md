# ffuf Content Discovery Optimization

Optimizing ffuf for fast, accurate content discovery without false positives.

## Basic Fuzzing

```bash
# Directory discovery
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt   -u https://target.com/FUZZ   -mc 200,204,301,302,307,401,403   -t 50   -recursion -recursion-depth 2
```

## Filtering False Positives

```bash
# Filter by response size (most common)
ffuf -w wordlist.txt -u https://target.com/FUZZ   -fs 0          # filter size 0 (empty)
  -fs 1234       # filter specific size
  -fw 2          # filter word count
  -fl 5          # filter line count

# Auto-calibrate filters
ffuf -w wordlist.txt -u https://target.com/FUZZ   -ac            # auto-calibrate from random responses
```

## Parameter Fuzzing

```bash
# GET parameters
ffuf -w params.txt -u "https://target.com/api?FUZZ=test" -mc 200

# POST body parameters
ffuf -w params.txt -X POST -d "FUZZ=test" -u https://target.com/api -mc 200

# JSON body
ffuf -w params.txt -X POST   -H "Content-Type: application/json"   -d '{"FUZZ":"test"}'   -u https://target.com/api
```

## vhost Discovery

```bash
ffuf -w subdomains.txt -u https://target.com   -H "Host: FUZZ.target.com"   -fs 0           # filter default page size
  -t 100
```

## Rate Limiting (Stealth)

```bash
ffuf -w wordlist.txt -u https://target.com/FUZZ   -rate 10        # 10 requests per second
  -p 0.1          # 100ms delay between requests
```

## Useful Wordlists

```bash
# SecLists paths
/usr/share/seclists/Discovery/Web-Content/
  raft-medium-directories.txt    # dirs
  raft-medium-files.txt          # files
  common.txt                     # quick scan
  quickhits.txt                  # common findings

# API endpoints
/usr/share/seclists/Discovery/Web-Content/api/
  api-endpoints.txt

# Parameters
/usr/share/seclists/Discovery/Web-Content/
  burp-parameter-names.txt
```
