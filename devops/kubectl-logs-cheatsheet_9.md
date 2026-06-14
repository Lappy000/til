# kubectl Logs Cheatsheet

```bash
# Follow logs
kubectl logs -f pod-name

# Previous container logs (crashed)
kubectl logs pod-name --previous

# Multi-container pod
kubectl logs pod-name -c container-name

# Last N lines
kubectl logs pod-name --tail=50

# Since timestamp
kubectl logs pod-name --since=1h
kubectl logs pod-name --since-time=2026-06-01T00:00:00Z

# All pods with label
kubectl logs -l app=web -f --max-log-requests=10

# Export to file
kubectl logs pod-name > /tmp/pod.log
```
