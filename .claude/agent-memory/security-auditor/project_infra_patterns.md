---
name: project-infra-patterns
description: Recurring security patterns and gaps observed in this project's Terraform — used to prioritize future audit focus areas
metadata:
  type: project
---

Infrastructure is a static S3 + CloudFront site with GitHub OIDC for CI/CD. Core access model is correct (OAC, not OAI; all S3 public-access blocked; HTTP redirected to HTTPS).

**Persistent findings to re-check on every audit:**

1. `terraform.tfstate` exists in the repo root with no `.gitignore`. State contains a real AWS account ID (`332120727902`) and full ARNs. The remote S3 backend in `backend.tf` is commented out, meaning state is local-only and at risk of being committed.

2. CloudFront `viewer_certificate` uses `cloudfront_default_certificate = true` with no `minimum_protocol_version` set, defaulting to TLSv1 (legacy). Should pin to `TLSv1.2_2021`.

3. No CloudFront response headers policy is attached to `default_cache_behavior`. Security headers (CSP, X-Frame-Options, HSTS, X-Content-Type-Options) are entirely absent.

4. No S3 server access logging or CloudFront access logging is configured anywhere. Zero audit trail for requests.

5. 404 error response is rewritten to HTTP 200 with `/index.html` — correct for SPA routing but masks real 404s in monitoring. Not a security issue, but worth noting for observability.

6. `project_name` default is `"hyejin"` — a real personal name used in bucket names and tags. Bucket name is predictable (`hyejin-<4-byte-hex>`). Low risk but reduces obscurity.

**Why:** These patterns appear in the first full audit of the repo. The state file commit risk is the most operationally dangerous gap.

**How to apply:** On subsequent audits, immediately check whether the state file is still present/committed, whether the remote backend has been enabled, and whether a response-headers policy has been added.
