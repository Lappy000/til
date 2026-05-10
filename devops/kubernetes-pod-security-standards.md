# Kubernetes Pod Security Standards: Practical Hardening Guide

Pod Security Standards (PSS) replaced PodSecurityPolicies (removed in K8s 1.25). Here's how to actually implement them without breaking your deployments.

## The Three Levels

| Level | What it blocks | Use case |
|---|---|---|
| **Privileged** | Nothing | System-level pods (CNI, storage drivers) |
| **Baseline** | Known privilege escalations | General workloads |
| **Restricted** | Anything non-essential | Hardened / multi-tenant clusters |

## Enforcement Modes

Apply via namespace labels — no CRDs or webhooks needed:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    # Enforce restricted: block non-compliant pods
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: latest
    # Warn on baseline violations (doesn't block, just warns)
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/audit: restricted
```

**Pro tip:** Start with `warn` mode to see what would break, then switch to `enforce`.

## Making Your Pods Compliant with "Restricted"

Most pods fail restricted mode. Here's the minimum securityContext you need:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: production
spec:
  template:
    spec:
      securityContext:
        runAsNonRoot: true
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: app
        image: myapp:latest
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          capabilities:
            drop: ["ALL"]
          seccompProfile:
            type: RuntimeDefault
        volumeMounts:
        - name: tmp
          mountPath: /tmp
      volumes:
      - name: tmp
        emptyDir: {}
```

## Common Breakages and Fixes

### 1. "readOnlyRootFilesystem breaks my app"

Many apps write to `/tmp` or log directories. Mount emptyDirs:

```yaml
volumes:
- name: tmp
  emptyDir:
    sizeLimit: 100Mi
```

### 2. "My image runs as root"

Build images properly or override at deploy time:

```dockerfile
# In Dockerfile
RUN adduser -D -u 1000 appuser
USER 1000
```

### 3. "I need NET_BIND_SERVICE for port 80"

Use a higher port and let the Service handle mapping:

```yaml
containers:
- name: web
  ports:
  - containerPort: 8080  # Not 80
---
apiVersion: v1
kind: Service
spec:
  ports:
  - port: 80
    targetPort: 8080
```

## Dry-Run Audit of Existing Cluster

```bash
# Label a namespace for warning mode first
kubectl label ns default \
  pod-security.kubernetes.io/warn=restricted \
  pod-security.kubernetes.io/warn-version=latest --overwrite

# Now restart pods and watch for warnings
kubectl delete pod --all -n default
```

## Admission Controller Alternatives

| Tool | Pros | Cons |
|---|---|---|
| **Kyverno** | K8s-native YAML policies | Another controller to manage |
| **OPA Gatekeeper** | Powerful Rego language | Steep learning curve |
| **Kubewarden** | Wasm-based, fast | Newer, smaller community |

## Key Takeaway

Label namespaces with `pod-security.kubernetes.io/warn: restricted` TODAY to see what breaks. Fix the easy stuff (securityContext, capabilities), then flip to `enforce`. This is the single highest-impact security improvement for most clusters.