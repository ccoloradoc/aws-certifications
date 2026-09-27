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

> Exam-wording cue: "encryption key usage must be **logged/auditable**" → rules out **AWS Owned Keys** (SSE-S3/SSE-SQS/default SSE-DDB) entirely — no CloudTrail visibility for you, since it's not even a key in your account. Both **AWS Managed Keys** and **Customer Managed Keys** satisfy this, since usage is logged in CloudTrail for both — the requirement alone doesn't tell you which of the two to pick; check the rotation and cross-account/key-policy requirements for that.

### By Cryptography

- **Symmetric (AES-256)** — a single key used to both encrypt and decrypt; what AWS services integrated with KMS use; you never get the key unencrypted — you must call the KMS API to use it
- **Asymmetric (RSA & ECC key pairs)** — public (encrypt) + private (decrypt) pair, for encrypt/decrypt or sign/verify; the public key is downloadable but the private key is never accessible unencrypted; use case: encryption outside AWS by users who can't call the KMS API

### Other Variants

- **Multi-Region keys** — see [Multi-Region Keys](#multi-region-keys)
- **Custom Key Store backed by CloudHSM** — see [KMS vs. CloudHSM](#kms-vs-cloudhsm)

> Exam-wording cue: "AWS service encrypts with KMS" → symmetric; "users outside AWS who can't call the KMS API need to encrypt data" or "sign/verify" → asymmetric. "Free, no control over the key policy" → AWS owned/managed; "control the key policy, rotation, or cross-account access" → customer managed.

> Exam-wording cue: "**encrypted at rest**," key rotation must happen "**automatically every 12 months**," "**cost-effective**," "**least operational overhead**" → **SSE-KMS with the AWS Managed Key (`aws/s3`)**. AWS Managed Keys already rotate automatically every year with **zero configuration and no monthly fee** — a **Customer Managed Key** can match the same rotation cadence, but only if you **manually enable** automatic rotation and pay its **$1/month** per-key charge, making it the higher-overhead, less cost-effective option when the requirement is satisfied by the managed key's default behavior alone. Don't over-reach for a CMK just because the requirement mentions "rotation" — check whether the *default* AWS Managed Key behavior already covers it before reaching for the option that needs manual setup.

## Key Rotation

- **AWS-managed keys** — rotate automatically every year
- **Customer-managed keys** — automatic rotation (must be enabled) or on-demand rotation
- **Imported keys** — manual rotation only, by swapping the alias to point at a new key

> Exam-wording cue: "encryption key usage must be logged" + "rotated every year" + "**MOST operationally efficient**" → **SSE-KMS with an AWS Managed Key** — it satisfies logging (CloudTrail) and yearly rotation with **zero configuration**, since AWS Managed Keys rotate automatically by default. A Customer Managed Key also technically satisfies both requirements, but costs more operational effort since **you** must explicitly enable rotation on it — don't default to CMK just because it sounds more "secure/controlled" when the question is only asking about logging + rotation, not about key-policy control or cross-account sharing (which *would* require a CMK, since AWS Managed Keys can't be shared cross-account).

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

> Exam-wording cue: "data must be encrypted client-side and **not disclosed even to the company's own admins**" → rules out server-side/SSE-KMS (DB admins can still read decrypted data through the engine) — the answer is **client-side encryption with the AWS Encryption SDK**, key access restricted via the KMS key policy to the app role only. Add "**worldwide customers**, **lowest latency**, multi-Region DB (Global Aurora/DynamoDB Global Tables)" → use a **KMS Multi-Region key** so each Region decrypts with a low-latency *local* KMS call instead of crossing Regions to a single-Region key.

> Exam-wording cue: "encrypted data replicated cross-region, but must use the **same encryption key** in every region" → **KMS Multi-Region Keys** — a standard (single-Region) KMS key is disqualified by definition, since it can't exist outside its own region; CRR's default behavior of re-encrypting with a separate destination-region key is exactly the behavior this requirement is ruling out. Crucially, a **single-Region key can never be converted/mutated into a Multi-Region key** — whether a key is single- or multi-Region is set permanently at creation and is immutable. The only valid path: **create a brand-new Multi-Region primary key + replica key**, then **decrypt the existing data and re-encrypt it under the new Multi-Region key** (e.g. via `ReEncrypt`, or an S3 Batch Operations job) — you cannot just "upgrade" the key already protecting the data in place. Any answer option describing "changing"/"converting" an existing key into a Multi-Region key is automatically wrong.

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

See [cloudhsm.md](cloudhsm.md) for the full comparison (single-tenant vs. multi-tenant, key access, HA, free tier). CloudHSM integrates with KMS through a Custom Key Store (EBS, S3, RDS, ...).

## To research

- Envelope encryption — how KMS encrypts data keys rather than data directly (slides only mention "could leverage Envelope Encryption" under client-side encryption)
- Grants vs. key policies for temporary/programmatic access (slides only show `CreateGrant` as a required permission)

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 301-330 (SSE-KMS limits), 631-660 (KMS core, key policies, Multi-Region keys, snapshot copying), and 661-690 (Multi-Region client-side encryption, S3 replication, AMI sharing).
