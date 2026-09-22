# Content Review Status

Tracks which topic files have been restructured into sections with exam-wording cues, which still have the old raw slide-dump format, and which haven't been reviewed yet. Generated 2026-09-22 by grepping every file for the old `## Answers (from slides` heading and for `Exam-wording cue` blockquotes.

## Legend

- ⚠️ **Old raw dump** — still has a `## Answers (from slides...)` section; needs restructuring into topic sections (see the editing conventions in [`CLAUDE.md`](../CLAUDE.md))
- ✅ **Restructured this session** — moved into sections, cues added/reviewed on 2026-09-19–22
- 🔲 **Has cues (earlier session)** — already restructured/cue'd in a prior session, not touched in this pass
- ⬜ **Not reviewed** — no exam-wording cues yet; may already be fine as prose, or may just not have been passed over yet

## Status Table

| File | Status | Cues | Open "To research" |
|---|---|---|---|
| 00-basics/aws-overview.md | ⬜ | 0 | — |
| 00-basics/global-infrastructure.md | ⬜ | 0 | — |
| 00-exam-strategy.md | ⬜ | 0 | — |
| 01-compute/containers-paas.md | ⬜ | 0 | — |
| 01-compute/ec2.md | ⬜ | 0 | — |
| 01-compute/lambda.md | ⚠️ | 0 | yes |
| 02-networking/cloudfront-and-global-accelerator.md | ⬜ | 0 | — |
| 02-networking/hybrid-connectivity.md | ⬜ | 0 | — |
| 02-networking/load-balancing.md | ⬜ | 0 | — |
| 02-networking/route53.md | ⬜ | 0 | — |
| 02-networking/vpc.md | ⬜ | 0 | — |
| 03-storage/comparison.md | ⬜ | 0 | — |
| 03-storage/ebs-instance-store.md | ⬜ | 0 | — |
| 03-storage/efs.md | ⬜ | 0 | — |
| 03-storage/fsx.md | ⬜ | 0 | — |
| 03-storage/glacier.md | ⬜ | 0 | — |
| 03-storage/s3.md | ⬜ | 0 | — |
| 03-storage/storage-gateway.md | ⬜ | 0 | — |
| 04-databases/1-rds.md | ⬜ | 0 | — |
| 04-databases/2-aurora.md | ⬜ | 0 | — |
| 04-databases/3-rds-proxy.md | ⬜ | 0 | — |
| 04-databases/4-elasticache.md | ⬜ | 0 | — |
| 04-databases/choosing-a-database.md | ⬜ | 0 | — |
| 04-databases/dynamodb.md | ⬜ | 0 | yes |
| 05-analytics/redshift-athena.md | 🔲 | 2 | — |
| 05-analytics/streaming-and-big-data.md | 🔲 | 6 | — |
| 06-application-integration/api-gateway.md | ⬜ | 0 | — |
| 06-application-integration/eventbridge.md | 🔲 | 1 | — |
| 06-application-integration/orchestration.md | ⬜ | 0 | — |
| 06-application-integration/sqs-sns.md | ⬜ | 0 | — |
| 07-security-identity/account-governance.md | ✅ | 2 | — |
| 07-security-identity/aws-certificate-manager.md | ✅ | 2 | — |
| 07-security-identity/aws-config.md | 🔲 | 1 | — |
| 07-security-identity/aws-organizations-and-control-tower.md | 🔲 | 1 | yes |
| 07-security-identity/cloudhsm.md | ✅ (new file, split out of waf-shield-firewall-manager.md) | 1 | — |
| 07-security-identity/cloudtrail.md | 🔲 | 1 | — |
| 07-security-identity/iam-sts-cognito.md | ⬜ | 0 | — |
| 07-security-identity/kms-deep-dive.md | ✅ | 3 | yes |
| 07-security-identity/monitoring-audit.md | 🔲 | 1 | — |
| 07-security-identity/secrets-management.md | ✅ | 1 | yes |
| 07-security-identity/shared-responsibility-model.md | ⬜ | 0 | yes |
| 07-security-identity/waf-shield-firewall-manager.md | ✅ | 2 | — |
| 08-management-governance/cicd-services.md | ⬜ | 0 | yes |
| 08-management-governance/cloudformation.md | ⬜ | 0 | — |
| 08-management-governance/cloudwatch.md | 🔲 | 1 | — |
| 08-management-governance/systems-manager.md | ⚠️ | 0 | yes |
| 08-management-governance/x-ray.md | ⬜ | 0 | — |
| 09-migration-transfer/migration-services.md | ⬜ | 0 | — |
| 09-migration-transfer/snow-family.md | ⬜ | 0 | yes |
| 10-resilience-dr/aws-backup.md | ⚠️ | 0 | yes |
| 10-resilience-dr/disaster-recovery.md | ⬜ | 0 | — |
| 10-resilience-dr/elastic-disaster-recovery.md | ⚠️ | 0 | yes |
| 11-other-services/hybrid-edge-infrastructure.md | ⚠️ | 0 | yes |
| 11-other-services/misc-managed-services.md | ⬜ | 0 | — |
| 12-cost-optimization/cost-management-tools.md | 🔲 | 1 | yes |
| 13-machine-learning/ai-services.md | 🔲 | 3 | — |
| well-architected-framework.md | ⚠️ | 0 | yes |

## Priority Queue

Files confirmed still in the old unstructured format (⚠️), highest-value to fix next:

1. `01-compute/lambda.md`
2. `08-management-governance/systems-manager.md`
3. `10-resilience-dr/aws-backup.md`
4. `10-resilience-dr/elastic-disaster-recovery.md`
5. `11-other-services/hybrid-edge-infrastructure.md`
6. `well-architected-framework.md`

## Notes

<!-- Your own notes go here. -->

This file is a snapshot, not auto-maintained — re-run the grep below and update the table after each restructuring pass:

```bash
for f in $(find . -name "*.md" -not -path "./README.md" -not -path "*/open-research-items.md" -not -path "*/exam-guide-mapping.md" -not -path "*/official-exam-guide.md" -not -path "*/content-review-status.md" | sort); do
  old=$(grep -c "^## Answers (from slides" "$f" 2>/dev/null)
  cues=$(grep -c "Exam-wording cue" "$f" 2>/dev/null)
  research=$(grep -c "^## To research" "$f" 2>/dev/null)
  echo "$f | old_pattern=$old | cues=$cues | to_research=$research"
done
```
