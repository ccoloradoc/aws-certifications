# KMS — Deep Dive

Follow-up to the CMK basics in [encryption.md](encryption.md) — the base cheat sheet didn't go past naming the key types.

## To research

- Key policies vs. IAM policies for controlling KMS key access
- Automatic key rotation (annual, for AWS-managed and customer-managed keys) vs. manual rotation
- Multi-Region keys — replicating a KMS key across regions for cross-region encrypted workloads
- Envelope encryption — how KMS encrypts data keys rather than data directly
- Grants vs. key policies for temporary/programmatic access
- KMS request quotas and how they can throttle high-throughput applications

## Notes

<!-- Your own notes go here. -->
