# Python requests Session Reuse

```python
# BAD - new TCP connection every time
for url in urls:
    requests.get(url)

# GOOD - reuses connection pool
session = requests.Session()
for url in urls:
    session.get(url)
```

Session reuse is 3-5x faster for bulk requests. Also lets you set default headers, auth, and retry logic once.
