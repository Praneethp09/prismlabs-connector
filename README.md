# Prism Labs Connector

Thin CloudFormation stack deployed in the customer's AWS account to enable secure cross-account access for the Prism Labs ML platform.

## What it deploys

- **InputBucket** — encrypted S3 bucket for customer data uploads
- **OutputBucket** — encrypted S3 bucket for Prism Labs results
- **CrossAccountRole** — IAM role scoped to `s3:GetObject` (input) and `s3:PutObject` (output), assumable only by Prism Labs with `sts:ExternalId`
- **ConnectorLambda** — health check function

## Parameters

| Parameter | Description |
|-----------|-------------|
| `PrismLabsAccountId` | AWS account ID of the Prism Labs platform |
| `TenantId` | Unique tenant ID provided during registration |
| `ConnectorVersion` | Version tag (default: 1.0.0) |

## Deploy

```bash
aws cloudformation deploy \
  --template-file connector-stack.yml \
  --stack-name prismlabs-connector \
  --parameter-overrides \
    PrismLabsAccountId=YOUR_PLATFORM_ACCOUNT_ID \
    TenantId=YOUR_TENANT_ID \
  --capabilities CAPABILITY_NAMED_IAM
```

## Security

- No public access — all buckets are block public access enabled
- Encryption at rest (AES-256) enforced
- Transport security (`aws:SecureTransport` deny on all S3 actions)
- Cross-account role uses `sts:ExternalId` to prevent confused deputy
- Role scoped to minimum required permissions (GetObject on input, PutObject on output)
