---
name: tf-conventions
description: Terraform code style and conventions observed in the terraform/ directory of this portfolio-site project
metadata:
  type: project
---

Terraform for this project lives in `terraform/` with the standard layout (main.tf, variables.tf, outputs.tf). Observed conventions in main.tf:

- 2-space indentation, `terraform fmt` compatible.
- Single-line `#` comments placed directly above a resource describing intent (the "why"), not the "what". Example: `# Encrypt objects at rest with SSE-S3 (AES256) managed keys.`
- S3 resources are grouped together in an S3 section, ordered: `aws_s3_bucket.site` -> public access block -> `aws_s3_bucket_ownership_controls.site` -> (encryption) -> bucket policy data source + policy.
- Bucket name pattern: `bucket = "${var.project_name}-${random_id.bucket_suffix.hex}"`.
- Resources reference each other via attributes (e.g. `aws_s3_bucket.site.id`, `aws_s3_bucket.site.arn`) rather than hardcoded names/ARNs.
- S3 access is private + OAC-based: ownership = `BucketOwnerEnforced`, public access fully blocked, bucket policy grants only the CloudFront service principal `s3:GetObject`.

**Why:** Static site (S3 + CloudFront + GitHub OIDC) where all infra changes go through Terraform; consistency makes diffs reviewable.
**How to apply:** When adding/editing resources in `terraform/`, match this grouping and comment style; place new S3 bucket sub-resources near the other `aws_s3_bucket_*` resources.
