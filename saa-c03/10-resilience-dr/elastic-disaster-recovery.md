# AWS Elastic Disaster Recovery (DRS)

Newer DR service not mentioned in the base cheat sheet — complements the strategies in [disaster-recovery.md](disaster-recovery.md).

## To research

- How DRS automates continuous block-level replication from on-premises or other clouds into AWS
- How it fits into the Pilot Light / Warm Standby spectrum already documented in [disaster-recovery.md](disaster-recovery.md) — DRS essentially automates a low-cost pilot-light-style setup
- Recovery Time Objective (RTO) / Recovery Point Objective (RPO) DRS is designed to hit
- Failback process to the original source infrastructure

## Answers (from slides, pages 721-870)

- Formerly "CloudEndure Disaster Recovery"; quickly recovers physical, virtual, and cloud-based servers into AWS via **continuous block-level replication** (not periodic snapshots) — this is what gives it a low RPO
- Example targets: critical databases (Oracle, MySQL, SQL Server) and enterprise apps (SAP); also positioned as ransomware-attack recovery
- Includes a **failback** path to return to the original source infrastructure once it's restored, rather than staying on AWS permanently
- Fits the automated, low-cost pilot-light pattern from disaster-recovery.md: a minimal footprint is kept warm, with full-scale recovery only spun up on an actual disaster

## Notes

<!-- Your own notes go here. -->
