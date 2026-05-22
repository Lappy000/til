# Tailscale Funnel + VPN Conflicts on Windows

When running Tailscale Funnel to expose a local service (e.g., Bedrock proxy on port 8069), third-party VPNs can break the tunnel.

## Problem

- Tailscale Funnel works: `https://desktop-xxx.taileff116.ts.net:443` → `localhost:8069`
- Third-party VPN (Happ, Windscribe, etc.) overrides DNS/routing
- Tailscale's DERP relay traffic gets misrouted through VPN tunnel
- Result: 502/timeout on Funnel URLs, but normal sites work fine

## Diagnosis

```bash
# Check if Tailscale can reach its coordination server
tailscale status
tailscale ping <peer-hostname>

# Check if Funnel is actually listening
curl -s https://your-node.taileff116.ts.net/health
```

## Fix

1. **Split tunnel the VPN** — exclude Tailscale's DERP IPs (100.64.0.0/10)
2. **Route priority** — set Tailscale interface metric lower than VPN
3. **Use direct LAN** — if both machines are on same network, skip Funnel:
   ```
   # Instead of Tailscale Funnel:
   http://192.168.1.137:8069/api/v1
   ```

## Workaround for RU ISP Throttling

If the real issue is ISP throttling Cloudflare (which Tailscale Funnel uses):
- Use DoH (DNS over HTTPS) to bypass DNS-level blocks
- Or connect via LAN IP directly when on same network
