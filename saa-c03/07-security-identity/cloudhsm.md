# AWS CloudHSM

Dedicated-hardware alternative to KMS for key management. Pairs with [kms-deep-dive.md](kms-deep-dive.md).

## Overview

- **KMS** — AWS manages the software for encryption; **CloudHSM** — AWS provisions dedicated encryption hardware (HSM = Hardware Security Module)
- You manage your own encryption keys entirely (not AWS)
- Tamper-resistant device, FIPS 140-2 Level 3 compliance
- Supports both symmetric and asymmetric encryption (SSL/TLS keys)
- No free tier; you must use the CloudHSM client software
- Redshift supports CloudHSM for database encryption and key management
- Good option to use with SSE-C encryption

## Access, HA & Integration

- **IAM permissions** — only to create/read/update/delete an HSM cluster; the CloudHSM software manages the keys and the users
- **High availability** — CloudHSM clusters are spread across multiple AZs, good for availability and durability
- **Integration with AWS services** — through KMS: configure a KMS **Custom Key Store** with CloudHSM (e.g. EBS, S3, RDS)

## Recovery & Durability

- Tamper resistance means an HSM **wipes its keys** if it detects a physical intrusion attempt — so key availability depends on having other copies
- **Token keys** (persistent keys from generate/import/unwrap) are synchronized across every HSM in the cluster; running **at least two HSMs** is what makes this durable — a single-HSM cluster has no live copy to fall back on
- **Session keys** are ephemeral, exist on a single HSM only, and aren't synchronized
- Clusters can be restored from a **backup**; clusters created from the same backup share a "backup history" (same cluster certificate), which is how a KMS custom key store can be repointed to a related cluster
- AWS never has access to your keys, so it can't recover them for you — losing every HSM and every backup means the protected data can't be decrypted
- Unverified: CloudHSM takes automatic encrypted backups stored by AWS (standard behavior, but retention and cross-Region copy details weren't confirmed against the docs)
- The slides only state that clusters are spread across multiple AZs for HA; this section comes from the AWS docs, not the deck

## CloudHSM vs. KMS

| Feature | AWS KMS | AWS CloudHSM |
|---|---|---|
| Tenancy | Multi-tenant | Single-tenant |
| Standard | FIPS 140-2 Level 3 | FIPS 140-2 Level 3 |
| Master keys | AWS owned CMK, AWS managed CMK, customer managed CMK | Customer managed CMK |
| Key types | Symmetric, asymmetric, digital signing | Symmetric, asymmetric, digital signing & hashing |
| Key accessibility | Multiple AWS Regions (keys can't be accessed outside the Region they were created in) | Deployed and managed in a VPC; can be shared across VPCs (VPC peering) |
| Cryptographic acceleration | None | SSL/TLS acceleration, Oracle TDE acceleration |
| Access & authentication | AWS IAM | You create users and manage their permissions |
| High availability | AWS-managed service | Add multiple HSMs across different AZs |
| Audit capability | CloudTrail, CloudWatch | CloudTrail, CloudWatch, MFA support |
| Free tier | Yes | No |

> Exam-wording cue: "AWS must not have access to the keys / single-tenant dedicated hardware / you manage keys yourself" → **CloudHSM**; "AWS-managed encryption keys with IAM integration" → **KMS**. See [kms-deep-dive.md](kms-deep-dive.md).

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 661-690 (CloudHSM, CloudHSM vs. KMS); moved out of waf-shield-firewall-manager.md. The Recovery & Durability section comes from the AWS docs, not the deck.
