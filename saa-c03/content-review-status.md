# Content Review Status

Tracks which topic files have been restructured into sections (vs. still holding a raw slide dump). Generated 2026-09-22, revised same day after two rounds of correction (see Detection Notes).

## Legend

- ✅ **Done** — restructured into topic sections, no raw-dump heading; either spot-checked directly or has a clean multi-section structure matching the pattern of a verified file
- ⚠️ **Pending** — still has a raw-dump heading (`### From slides (pages ...)` or `## Answers (from slides ...)`, at any heading level) that needs to be split into proper topic sections
- A `## To research` section is **not** a pending signal — it's the current convention for genuinely unanswered gaps and appears in otherwise-finished files (e.g. `kms-deep-dive.md`)

## Detection Notes (so this doesn't regress again)

1. First pass used exam-wording-cue count as the completion signal — wrong. Cues are only added where the exam genuinely confuses two services, not on every finished file (`ec2.md`, `route53.md` have zero cues and are fully done).
2. Second pass grepped only `^## Answers (from slides` — missed `orchestration.md`, whose dump lived under a `### From slides (pages...)` heading nested inside `## Notes`.
3. Third pass grepped `From slides (pages|Answers (from slides` **and** `To research` together — over-flagged files like `kms-deep-dive.md`/`secrets-management.md` that have a legitimate, current-convention `## To research` section but no actual dump.
4. **Current method**: grep only for the two literal dump-heading phrases (no `To research`), then cross-check with `## ` heading count/spot-reads. Route53 and ec2 were manually confirmed by the user as the reference pattern for "done": several topical `##` sections, then `## Notes` with Netec blockquotes and/or a provenance line.

```bash
for f in $(find . -name "*.md" -not -path "./README.md" -not -path "*/open-research-items.md" -not -path "*/exam-guide-mapping.md" -not -path "*/official-exam-guide.md" -not -path "*/content-review-status.md" | sort); do
  dump=$(grep -c -i "From slides (pages\|Answers (from slides" "$f" 2>/dev/null)
  echo "$f | dump_headings=$dump"
done
```

## Status Table

| File | Status |
|---|---|
| 00-basics/aws-overview.md | ✅ |
| 00-basics/global-infrastructure.md | ✅ |
| 00-exam-strategy.md | ✅ |
| 01-compute/containers-paas.md | ✅ |
| 01-compute/ec2.md | ✅ (user-confirmed reference pattern) |
| 01-compute/lambda.md | ⚠️ |
| 02-networking/cloudfront-and-global-accelerator.md | ✅ |
| 02-networking/hybrid-connectivity.md | ✅ |
| 02-networking/load-balancing.md | ✅ (spot-checked) |
| 02-networking/route53.md | ✅ (user-confirmed reference pattern) |
| 02-networking/vpc.md | ✅ (spot-checked headings) |
| 03-storage/comparison.md | ✅ (spot-checked — intentional index table, not a topic file) |
| 03-storage/ebs-instance-store.md | ⚠️ |
| 03-storage/efs.md | ⚠️ |
| 03-storage/fsx.md | ✅ |
| 03-storage/glacier.md | ⚠️ |
| 03-storage/s3.md | ✅ (spot-checked headings) |
| 03-storage/storage-gateway.md | ✅ |
| 04-databases/1-rds.md | ✅ |
| 04-databases/2-aurora.md | ✅ |
| 04-databases/3-rds-proxy.md | ✅ |
| 04-databases/4-elasticache.md | ✅ |
| 04-databases/choosing-a-database.md | ✅ |
| 04-databases/dynamodb.md | ✅ (has `## To research` only, not a dump) |
| 05-analytics/redshift-athena.md | ✅ |
| 05-analytics/streaming-and-big-data.md | ✅ |
| 06-application-integration/api-gateway.md | ✅ |
| 06-application-integration/eventbridge.md | ✅ (spot-checked) |
| 06-application-integration/orchestration.md | ✅ (restructured this session) |
| 06-application-integration/sqs-sns.md | ✅ |
| 07-security-identity/account-governance.md | ✅ |
| 07-security-identity/aws-certificate-manager.md | ✅ |
| 07-security-identity/aws-config.md | ✅ (spot-checked) |
| 07-security-identity/aws-organizations-and-control-tower.md | ✅ (has `## To research` only, not a dump) |
| 07-security-identity/cloudhsm.md | ✅ (new file, split out of waf-shield-firewall-manager.md) |
| 07-security-identity/cloudtrail.md | ✅ (spot-checked) |
| 07-security-identity/iam-sts-cognito.md | ⚠️ |
| 07-security-identity/kms-deep-dive.md | ✅ (has `## To research` only, not a dump) |
| 07-security-identity/monitoring-audit.md | ✅ |
| 07-security-identity/secrets-management.md | ✅ (has `## To research` only, not a dump) |
| 07-security-identity/shared-responsibility-model.md | ✅ (has `## To research` only, not a dump) |
| 07-security-identity/waf-shield-firewall-manager.md | ✅ |
| 08-management-governance/cicd-services.md | ✅ (has `## To research` only, not a dump) |
| 08-management-governance/cloudformation.md | ⚠️ |
| 08-management-governance/cloudwatch.md | ✅ |
| 08-management-governance/systems-manager.md | ⚠️ |
| 08-management-governance/x-ray.md | ✅ (thin by design — slides cover it in one line only; correctly left as a `## To research` gap, nothing to restructure) |
| 09-migration-transfer/migration-services.md | ✅ (spot-checked headings) |
| 09-migration-transfer/snow-family.md | ✅ (has `## To research` only, not a dump) |
| 10-resilience-dr/aws-backup.md | ⚠️ |
| 10-resilience-dr/disaster-recovery.md | ⚠️ |
| 10-resilience-dr/elastic-disaster-recovery.md | ⚠️ |
| 11-other-services/hybrid-edge-infrastructure.md | ⚠️ |
| 11-other-services/misc-managed-services.md | ⚠️ (spot-checked — confirmed genuine dump) |
| 12-cost-optimization/cost-management-tools.md | ✅ (has `## To research` only, not a dump) |
| 13-machine-learning/ai-services.md | ✅ (spot-checked headings) |
| well-architected-framework.md | ⚠️ |

## Priority Queue

Confirmed still holding a raw-dump heading — the only files that actually need restructuring:

1. `01-compute/lambda.md`
2. `03-storage/ebs-instance-store.md`
3. `03-storage/efs.md`
4. `03-storage/glacier.md`
5. `07-security-identity/iam-sts-cognito.md`
6. `08-management-governance/cloudformation.md`
7. `08-management-governance/systems-manager.md`
8. `10-resilience-dr/aws-backup.md`
9. `10-resilience-dr/disaster-recovery.md`
10. `10-resilience-dr/elastic-disaster-recovery.md`
11. `11-other-services/hybrid-edge-infrastructure.md`
12. `11-other-services/misc-managed-services.md`
13. `well-architected-framework.md`

## Notes

<!-- Your own notes go here. -->

This file is a snapshot, not auto-maintained — re-run the grep above after each restructuring pass and flip the row from ⚠️ to ✅.
