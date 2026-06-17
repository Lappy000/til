# terraform import Existing Resources

```bash
# Import an existing AWS instance
terraform import aws_instance.web i-1234567890abcdef0

# Import S3 bucket
terraform import aws_s3_bucket.data my-bucket-name

# Import with for_each (complex)
terraform import 'aws_instance.web["key"]' i-1234567890abcdef0

# After import, run plan to see drift
terraform plan
# Will show what's missing from your .tf file
```

Import brings the resource into state but does NOT generate the .tf config. Use `terraform show` to inspect and manually write the resource block.
