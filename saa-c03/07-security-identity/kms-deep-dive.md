# KMS — Deep Dive

Follow-up to the CMK basics in [encryption.md](encryption.md) — the base cheat sheet didn't go past naming the key types.

## To research

- Key policies vs. IAM policies for controlling KMS key access
- Automatic key rotation (annual, for AWS-managed and customer-managed keys) vs. manual rotation
- Multi-Region keys — replicating a KMS key across regions for cross-region encrypted workloads
- Envelope encryption — how KMS encrypts data keys rather than data directly
- Grants vs. key policies for temporary/programmatic access
- KMS request quotas and how they can throttle high-throughput applications

## Answers (from slides, pages 571-720)

- KMS Key types: Symmetric (AES-256, single key for encrypt+decrypt, never retrievable unencrypted — you always call the KMS API to use it; what most AWS service integrations use) vs. Asymmetric (RSA/ECC public+private pair, for encrypt/decrypt or sign/verify, public key downloadable, private key never exposed — used when the caller can't reach the KMS API directly)
- Key ownership tiers: AWS Owned Keys (free — SSE-S3, SSE-SQS, default SSE-DDB), AWS Managed Keys (free, `aws/service-name` alias), Customer Managed Keys — created or imported ($1/month + $0.03/10,000 API calls)
- Rotation: AWS-managed keys rotate automatically every year; customer-managed keys support automatic (must be enabled) or on-demand rotation; imported keys only support manual rotation via alias swap
- **Key Policies** — like S3 bucket policies but mandatory: you *cannot* control access to a KMS key without one. Default policy grants the account root full access; a custom policy defines who can use/administer the key and enables cross-account access
- Cross-account/cross-region snapshot copying pattern: encrypt the snapshot with your own CMK → attach a key policy authorizing the target account/role → share the snapshot → target account copies it, re-encrypting with a CMK in its own account → create a volume from the copy. Cross-account requires the target IAM role to have `DescribeKey`, `ReEncrypt*`, `CreateGrant`, `Decrypt` on the source key
- **Multi-Region Keys** — identical key ID/material/rotation across regions, so you can encrypt in one region and decrypt in another with no re-encryption or cross-region API calls; each regional copy (primary + replicas) is still managed independently — they are NOT a single global key. Use cases: global client-side encryption, encrypting Global DynamoDB Tables or Global Aurora, so each region's clients can decrypt via a low-latency *local* KMS call instead of crossing regions
- S3 replication + SSE-KMS: unencrypted and SSE-S3 objects replicate by default; SSE-C can be replicated; SSE-KMS objects require explicitly enabling KMS replication, picking a destination-bucket KMS key, adjusting that key's policy, and giving the replication IAM role `kms:Decrypt` (source) + `kms:Encrypt` (destination) — watch for KMS throttling under heavy replication; multi-region keys don't skip the decrypt/re-encrypt step in S3 (S3 still treats each regional copy as independent)

## Notes

<!-- Your own notes go here. -->
