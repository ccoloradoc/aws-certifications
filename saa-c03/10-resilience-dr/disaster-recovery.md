# Disaster Recovery (DR)

## Approaches (increasing cost & complexity)

1. **Backup and Restore** — lowest cost, slowest recovery
2. **Pilot Light** — small core services always running, scale up on failover
3. **Warm Standby** — scaled-down but fully functional environment running at all times
4. **Multi-site (Active-Active)** — on-premises and AWS both actively serving traffic

## Cross-Region DR

- Copy an AMI to a secondary region to enable recovery there

## Notes

<!-- Your own notes go here. -->

### From slides (pages 721-870)

- **RPO** (Recovery Point Objective) — how much data loss (measured in time) is acceptable; **RTO** (Recovery Time Objective) — how long recovery is allowed to take. Cost/complexity increases from Backup & Restore (high RPO/RTO, cheapest) through Pilot Light → Warm Standby → Multi-Site/Hot Site (near-zero RTO, most expensive, full production scale running in both places)
- Warm Standby specifically: a full, functional environment always running, but at minimum size — scale it up to production load on failover

> Exam-wording cue: "a scaled-down version of a fully functional environment is always running," "recovery time kept to a minimum" → Warm Standby. "Fully functional" is the word that rules out Pilot Light (only core/critical components like the database run continuously — the rest of the stack must be launched from scratch on failover, so it isn't "fully functional" while idle). "Always running" rules out Backup and Restore (nothing runs until disaster strikes). Not running at full production scale rules out Multi-Site/Hot Standby (full scale in both locations continuously) — Warm Standby is specifically the middle ground: the whole stack exists and runs, just smaller, with recovery being a scale-up rather than a build-from-scratch or full-cost duplication.
- Categories of DR groundwork, from the slides' recap:
  - **Backup** — EBS snapshots, RDS automated backups/snapshots, S3/S3-IA/Glacier lifecycle pushes + Cross-Region Replication, Snowball/Storage Gateway from on-prem
  - **High Availability** — Route 53 for DNS failover across regions, RDS/ElastiCache Multi-AZ, EFS, S3, Site-to-Site VPN as a DX fallback
  - **Replication** — RDS Cross-Region read replicas, Aurora Global Database, on-prem→RDS replication, Storage Gateway
  - **Automation** — CloudFormation/Elastic Beanstalk to rebuild an environment from scratch, CloudWatch-triggered EC2 recovery/reboot, Lambda for custom automation
  - **Chaos engineering** — deliberately breaking things (Netflix's "Simian Army") to validate resilience before a real disaster does it for you
- For bulk data-transfer time estimates when standing up a DR region (internet/VPN vs. Direct Connect vs. Snowball), see [migration-services.md](../09-migration-transfer/migration-services.md)
