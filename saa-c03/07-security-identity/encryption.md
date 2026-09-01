# Encryption

## Data at Rest

### Client-side

- KMS-managed Customer Master Key (CMK)
- Application-managed master key

### Server-side

- **SSE-C** — customer-provided key, managed on-premises
- **SSE-S3** — Amazon-managed key
- **SSE-KMS** — AWS Key Management Service-managed key
- **CloudHSM** — user-generated keys, held in a dedicated cloud HSM (see [waf-shield-firewall-manager.md](waf-shield-firewall-manager.md) for the detailed CloudHSM vs. KMS comparison — filed there since that's where the slides covered it)

## Data in Motion

- SSL/TLS for transit encryption (HTTPS, port 443)

## Best Practices

- Lock or delete root user access keys
- Never store access keys in S3

## Notes

<!-- Your own notes go here. -->
