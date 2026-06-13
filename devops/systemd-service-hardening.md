# systemd Service Hardening with Sandboxing

Locking down systemd services using built-in sandboxing directives.

## Hardened Service File

```ini
[Unit]
Description=Web Application
After=network.target

[Service]
Type=simple
User=webapp
Group=webapp
WorkingDirectory=/opt/webapp
ExecStart=/opt/webapp/bin/server

# --- Resource Limits ---
MemoryMax=512M
CPUQuota=200%
TasksMax=64
LimitNOFILE=4096

# --- Filesystem ---
ProtectSystem=strict
ProtectHome=true
PrivateTmp=true
ReadWritePaths=/opt/webapp/data /var/log/webapp
NoNewPrivileges=true

# --- Network ---
RestrictAddressFamilies=AF_INET AF_INET6 AF_UNIX
IPAddressDeny=any
IPAddressAllow=localhost

# --- Kernel ---
ProtectKernelTunables=true
ProtectKernelModules=true
ProtectKernelLogs=true
ProtectControlGroups=true
ProtectClock=true
ProtectHostname=true

# --- Capabilities ---
CapabilityBoundingSet=
AmbientCapabilities=
SystemCallFilter=@system-service
SystemCallFilter=~@privileged @resources @mount
SystemCallArchitectures=native

# --- Misc ---
LockPersonality=true
RestrictRealtime=true
RestrictSUIDSGID=true
RemoveIPC=true
UMask=0077

# --- Restart ---
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

## Verify with systemd-analyze

```bash
# Check security score
systemd-analyze security webapp.service

# View effective sandboxing
systemd-analyze cat-config systemd/webapp.service

# Test service file syntax
systemd-analyze verify /etc/systemd/system/webapp.service
```

## Gradual Rollout

```bash
# 1. Apply non-breaking settings first
systemctl daemon-reload
systemctl restart webapp

# 2. Add restrictions one by one
# 3. Check for denials in journal
journalctl -u webapp -f
# If service breaks, remove the last directive added
```
