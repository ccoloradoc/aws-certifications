# AWS Backup

Not mentioned in the base cheat sheet, despite individual services (EBS Data Lifecycle Manager, RDS snapshots) being covered piecemeal elsewhere.

## To research

- Centralized backup policies across EBS, RDS, DynamoDB, EFS, and other services
- Backup plans and lifecycle rules (transition to cold storage, expiration)
- Cross-region and cross-account backup copying
- Backup vaults and vault lock (compliance mode)
- How this consolidates what was previously scattered per-service (compare with [ebs-instance-store.md](../03-storage/ebs-instance-store.md) and [rds-aurora.md](../04-databases/rds-aurora.md))

## Answers (from slides, pages 721-870)

- Fully managed, centralizes backup automation so you don't write custom scripts per-service
- Supported services: EC2/EBS, S3, RDS (all engines)/Aurora/DynamoDB, DocumentDB, Neptune, EFS, FSx (Lustre & Windows), and Storage Gateway (Volume Gateway)
- Supports cross-region and cross-account backup copies, and PITR where the underlying service supports it
- **Backup Plans** define: frequency (every 12h, daily, weekly, monthly, or a cron expression), a backup window, on-demand or scheduled triggers, tag-based selection of which resources to back up, transition-to-cold-storage timing, and retention period
- **Backup Vault Lock** — enforces WORM on everything in a vault: blocks deletion and blocks shortening/altering retention periods, even for the root user, once enabled — a hard compliance guarantee beyond a normal IAM Deny

## Notes

<!-- Your own notes go here. -->
