# GitHub Actions pull_request_target Pwn Request

## Vulnerable Pattern

```yaml
on: pull_request_target
jobs:
  build:
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}
      - run: npm install  # Executes attacker code with secrets!
```

## Attack Chain

1. Attacker opens PR from fork
2. Workflow runs with base repo secrets
3. checkout fetches attacker branch
4. Build step runs attacker code
5. Secrets exfiltrated

## Real Impact: CVE-2026-45321

TanStack compromise used this + cache poisoning + OIDC extraction.
42 packages compromised, 84 malicious versions.

## Fix

Never checkout PR head in pull_request_target. Use `pull_request` trigger instead.
