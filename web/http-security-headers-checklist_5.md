# HTTP Security Headers Checklist

```bash
# Quick check with curl
curl -sI https://target.com | grep -iE 'strict|x-frame|x-content|content-security|referrer|permissions'
```

Essential headers:
```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Content-Security-Policy: default-src 'self'
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=(), camera=()
```

```bash
# Scan with securityheaders.com
curl -sI https://target.com | head -30
```
