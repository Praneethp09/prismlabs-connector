# Prism Labs Connector

Thin CloudFormation stack deployed in the customer's AWS account to enable secure cross-account access for the Prism Labs ML platform.

## What it deploys

- **InputBucket** — encrypted S3 bucket for customer data uploads
- **OutputBucket** — encrypted S3 bucket for Prism Labs results
- **CrossAccountRole** — IAM role for S3 read/write + Glue catalog read, assumable only by Prism Labs with `sts:ExternalId`
- **GlueCatalogShare** — RAM resource share that gives the platform read-only access to the customer's Glue catalog (so FE users can query customer's Athena tables from the Prism Labs UI)
- **ConnectorLambda** — health check function

## Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `PrismLabsAccountId` | AWS account ID of the Prism Labs platform | `536784880613` |
| `TenantId` | Unique tenant ID provided during registration | — |
| `ConnectorVersion` | Version tag | `1.1.0` |

## Deploy

```bash
aws cloudformation deploy \
  --template-file connector-stack.yml \
  --stack-name prismlabs-connector-{YOUR_TENANT_ID} \
  --parameter-overrides \
    TenantId=YOUR_TENANT_ID \
  --capabilities CAPABILITY_NAMED_IAM
```

The `PrismLabsAccountId` parameter defaults to the platform account. Only override if deploying for a different platform environment.

## Outputs

| Output | Description |
|--------|-------------|
| `CrossAccountRoleArn` | ARN of the cross-account IAM role |
| `InputBucketName` | S3 bucket for uploading input data |
| `OutputBucketName` | S3 bucket for receiving results |
| `GlueCatalogArn` | ARN of the shared Glue catalog |
| `ConnectorVersion` | Stack version |

## Security

- No public access — all buckets are block public access enabled
- Encryption at rest (AES-256) enforced
- Transport security (`aws:SecureTransport` deny on all S3 actions)
- Cross-account role uses `sts:ExternalId` to prevent confused deputy
- Role scoped to minimum required permissions (GetObject on input, PutObject on output)
- Glue catalog shared via RAM with read-only access — platform can query tables, cannot create/drop
