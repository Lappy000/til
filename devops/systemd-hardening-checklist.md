# systemd Service Hardening

## Essential Directives

```ini
[Service]
ProtectSystem=strict
ProtectHome=true
PrivateTmp=true
NoNewPrivileges=true
PrivateDevices=true
ProtectKernelTunables=true
ProtectKernelModules=true
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
SystemCallFilter=@system-service
SystemCallArchitectures=native
MemoryMax=512M
CPUQuota=50%
TasksMax=64
```

## Audit

```bash
systemd-analyze security myapp.service
# Score 0-10 (lower = better)
```

## Dynamic User (no useradd)

```ini
DynamicUser=true
StateDirectory=myapp
```
