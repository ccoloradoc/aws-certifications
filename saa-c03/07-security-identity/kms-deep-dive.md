# KMS — Deep Dive

Follow-up to the CMK basics mentioned in [s3.md](../03-storage/s3.md)'s Client-Side Encryption bullet — the base cheat sheet didn't go past naming the key types.

## Overview

- "Anytime you hear 'encryption' for an AWS service, it's most likely KMS" — AWS manages the keys for you
- Fully integrated with IAM for authorization; key usage is auditable via CloudTrail
- Seamlessly integrated into most AWS services (EBS, S3, RDS, SSM, ...)
- Also usable through API calls (SDK, CLI) — e.g. to encrypt secrets stored in code/environment variables; never store secrets in plaintext, especially in code

## Types of KMS Keys

Keys are classified along two independent axes: **who owns/manages them** and **what cryptography they use**.

### By Ownership

| Type | Cost | Notes |
|---|---|---|
| **AWS Owned Keys** | Free | Used by SSE-S3, SSE-SQS, and the default SSE-DDB key; owned by the AWS service and shared across many customer accounts; not visible in your account, no key policy control, no CloudTrail logging for you |
| **AWS Managed Keys** | No monthly fee (per-use API charges apply) | Alias `aws/service-name` (e.g. `aws/rds`, `aws/ebs`, `aws/dynamodb`); a real key in your account, unique per account + service + Region; key policy is service-controlled (viewable, not editable); usage logged in CloudTrail; rotated automatically every year |
| **Customer Managed Keys (CMK)** — created in KMS | $1/month | You control the key policy; rotation must be enabled (automatic or on-demand) |
| **Customer Managed Keys (CMK)** — imported | $1/month | You supply the key material; manual rotation only (alias swap) |

- **Owned vs. managed**: owned = you never see the key; managed = you can see and audit it, but can't control its policy or rotation
- **AWS managed keys can't be shared cross-account** — their key policy can't be edited to trust another account, so cross-account sharing of encrypted resources (snapshots, AMIs) requires a customer managed key; they're also Regional, so cross-Region copies need re-encryption with a key in the destination Region
- All customer-managed keys also cost $0.03 per 10,000 KMS API calls
- **KMS Keys** is the new name for KMS Customer Master Key (CMK)
- See [Key Rotation](#key-rotation) for the rotation details per type

### By Cryptography

- **Symmetric (AES-256)** — a single key used to both encrypt and decrypt; what AWS services integrated with KMS use; you never get the key unencrypted — you must call the KMS API to use it
- **Asymmetric (RSA & ECC key pairs)** — public (encrypt) + private (decrypt) pair, for encrypt/decrypt or sign/verify; the public key is downloadable but the private key is never accessible unencrypted; use case: encryption outside AWS by users who can't call the KMS API

### Other Variants

- **Multi-Region keys** — see [Multi-Region Keys](#multi-region-keys)
- **Custom Key Store backed by CloudHSM** — see [KMS vs. CloudHSM](#kms-vs-cloudhsm)

> Exam-wording cue: "AWS service encrypts with KMS" → symmetric; "users outside AWS who can't call the KMS API need to encrypt data" or "sign/verify" → asymmetric. "Free, no control over the key policy" → AWS owned/managed; "control the key policy, rotation, or cross-account access" → customer managed.

## Key Rotation

- **AWS-managed keys** — rotate automatically every year
- **Customer-managed keys** — automatic rotation (must be enabled) or on-demand rotation
- **Imported keys** — manual rotation only, by swapping the alias to point at a new key

## Key Policies

- Control access to KMS keys, "similar" to S3 bucket policies — but mandatory: you *cannot* control access to a KMS key without one
- **Default key policy** — created if you don't provide one; gives the account root user (= the entire AWS account) complete access to the key
- **Custom key policy** — defines which users/roles can use the key and who can administer it; useful for cross-account access to your key

## Multi-Region Keys

- Identical KMS keys in different Regions, usable interchangeably — same key ID, key material, automatic rotation, etc.
- Encrypt in one Region and decrypt in another, with no re-encryption and no cross-Region API calls
- **NOT global** — a primary plus replicas, and each Multi-Region key is managed independently
- Use cases: global client-side encryption, encryption on Global DynamoDB Tables or Global Aurora
- **DynamoDB Global Tables + client-side encryption** — encrypt specific attributes with the Amazon DynamoDB Encryption Client; the encrypted data replicates to other Regions, and clients there decrypt with low-latency *local* KMS calls using the replicated multi-Region key
- **Global Aurora + client-side encryption** — same pattern using the AWS Encryption SDK; protects specific fields even from database admins, since decryption requires access to the key

## Service Integration Patterns

### SSE-KMS on S3

- Adds user control + auditing (CloudTrail) over SSE-S3; header `x-amz-server-side-encryption: aws:kms`
- **Limitation** — upload calls `GenerateDataKey` and download calls `Decrypt`, both counting toward the KMS request quota per second (5,500, 10,000, or 30,000 req/s depending on Region); request an increase via the Service Quotas console

### S3 Replication with SSE-KMS

- Unencrypted and SSE-S3 objects replicate by default; SSE-C objects can be replicated
- SSE-KMS objects require explicitly enabling the option, then: pick the KMS key for the destination bucket, adapt that key's policy, and give the replication IAM role `kms:Decrypt` (source key) + `kms:Encrypt` (destination key)
- Heavy replication can hit KMS throttling — ask for a Service Quotas increase
- Multi-Region keys can be used, but S3 treats each regional copy as an independent key (the object is still decrypted and re-encrypted)

### EBS Snapshots — Cross-Account / Cross-Region Copy

- **Across accounts**: encrypt the snapshot with your own CMK → attach a key policy authorizing the target account/role → share the snapshot → target account copies it, re-encrypting with a CMK in its own account → create a volume from the copy
- **Across Regions**: the copy is re-encrypted (KMS ReEncrypt) with a key in the destination Region

### AMI Sharing (Encrypted)

1. AMI in the source account is encrypted with a source-account KMS key
2. Modify the image attribute to add a Launch Permission for the target account
3. Share the KMS key(s) used to encrypt the referenced snapshot with the target account/role
4. The target IAM role/user needs `DescribeKey`, `ReEncrypt*`, `CreateGrant`, `Decrypt`
5. On launch, the target account can optionally re-encrypt the volumes with a new KMS key in its own account

> Exam-wording cue: "share an encrypted AMI with another account" → three things, all required: (1) add the target account to the AMI's **Launch Permission**, (2) **share the KMS key** that encrypted the snapshot (key policy/grant to the target account or role), (3) give the target IAM role/user `DescribeKey`, `ReEncrypt*`, `CreateGrant`, `Decrypt`. It must be a **customer managed key** — an AWS managed key (`aws/ebs`) can't be shared. Re-encrypting on launch with a key in the target account is optional. "Target can't launch the shared encrypted AMI" → the KMS key wasn't shared or the role lacks those KMS permissions.

## KMS vs. CloudHSM

See [waf-shield-firewall-manager.md](waf-shield-firewall-manager.md) for the full comparison (single-tenant vs. multi-tenant, key access, HA, free tier). CloudHSM integrates with KMS through a Custom Key Store (EBS, S3, RDS, ...).

## To research

- Envelope encryption — how KMS encrypts data keys rather than data directly (slides only mention "could leverage Envelope Encryption" under client-side encryption)
- Grants vs. key policies for temporary/programmatic access (slides only show `CreateGrant` as a required permission)

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 301-330 (SSE-KMS limits), 631-660 (KMS core, key policies, Multi-Region keys, snapshot copying), and 661-690 (Multi-Region client-side encryption, S3 replication, AMI sharing, CloudHSM vs. KMS).
