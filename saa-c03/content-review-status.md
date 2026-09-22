# Content Review Status

Tracks which topic files have been restructured into sections (vs. still holding a raw slide dump), and which have exam-wording cues. Generated 2026-09-22, updated same day after the detection method was corrected (see below).

## Legend

- ✅ **Confirmed done** — restructured into topic sections; either edited/verified in a session, or visually confirmed with no raw-dump heading left
- ⚠️ **Pending** — still has a raw-dump heading (`### From slides (pages ...)` or `## Answers (from slides ...)`, at any heading level) that needs to be split into proper topic sections
- ⬜ **Unverified** — no raw-dump heading detected, but not yet manually opened/confirmed; likely fine, just not checked

**Detection note:** the first version of this table used exam-wording-cue count as a proxy for "done," which was wrong — `ec2.md` is fully restructured with zero cues (cues are only added where the exam genuinely confuses two services, not on every file), while `orchestration.md` had a raw dump under a `### From slides (pages...)` heading nested inside `## Notes` that a `^## Answers (from slides` -only grep missed. The table below instead greps for either dump-heading pattern at any level.

## Status Table

| File | Status | Cues | Open "To research" |
|---|---|---|---|
| 00-basics/aws-overview.md | ⬜ | 0 | — |
| 00-basics/global-infrastructure.md | ⬜ | 0 | — |
| 00-exam-strategy.md | ⬜ | 0 | — |
| 01-compute/containers-paas.md | ⬜ | 0 | — |
| 01-compute/ec2.md | ✅ (verified — sections + Netec notes, no dump) | 0 | — |
| 01-compute/lambda.md | ⚠️ | 0 | yes |
| 02-networking/cloudfront-and-global-accelerator.md | ⬜ | 0 | — |
| 02-networking/hybrid-connectivity.md | ⬜ | 0 | — |
| 02-networking/load-balancing.md | ⬜ | 0 | — |
| 02-networking/route53.md | ⬜ | 0 | — |
| 02-networking/vpc.md | ⬜ | 0 | — |
| 03-storage/comparison.md | ⬜ | 0 | — |
| 03-storage/ebs-instance-store.md | ⚠️ | 0 | — |
| 03-storage/efs.md | ⚠️ | 0 | — |
| 03-storage/fsx.md | ⬜ | 0 | — |
| 03-storage/glacier.md | ⚠️ | 0 | — |
| 03-storage/s3.md | ⬜ | 0 | — |
| 03-storage/storage-gateway.md | ⬜ | 0 | — |
| 04-databases/1-rds.md | ⬜ | 0 | — |
| 04-databases/2-aurora.md | ⬜ | 0 | — |
| 04-databases/3-rds-proxy.md | ⬜ | 0 | — |
| 04-databases/4-elasticache.md | ⬜ | 0 | — |
| 04-databases/choosing-a-database.md | ⬜ | 0 | — |
| 04-databases/dynamodb.md | ⬜ | 0 | yes |
| 05-analytics/redshift-athena.md | ⬜ | 2 | — |
| 05-analytics/streaming-and-big-data.md | ⬜ | 6 | — |
| 06-application-integration/api-gateway.md | ⬜ | 0 | — |
| 06-application-integration/eventbridge.md | ⬜ | 1 | — |
| 06-application-integration/orchestration.md | ✅ (restructured this session) | 0 | — |
| 06-application-integration/sqs-sns.md | ⬜ | 0 | — |
| 07-security-identity/account-governance.md | ✅ | 2 | — |
| 07-security-identity/aws-certificate-manager.md | ✅ | 2 | — |
| 07-security-identity/aws-config.md | ⬜ | 1 | — |
| 07-security-identity/aws-organizations-and-control-tower.md | ⬜ | 1 | yes |
| 07-security-identity/cloudhsm.md | ✅ (new file, split out of waf-shield-firewall-manager.md) | 1 | — |
| 07-security-identity/cloudtrail.md | ⬜ | 1 | — |
| 07-security-identity/iam-sts-cognito.md | ⚠️ | 0 | — |
| 07-security-identity/kms-deep-dive.md | ✅ | 3 | yes |
| 07-security-identity/monitoring-audit.md | ⬜ | 1 | — |
| 07-security-identity/secrets-management.md | ✅ | 1 | yes |
| 07-security-identity/shared-responsibility-model.md | ⬜ | 0 | yes |
| 07-security-identity/waf-shield-firewall-manager.md | ✅ | 2 | — |
| 08-management-governance/cicd-services.md | ⬜ | 0 | yes |
| 08-management-governance/cloudformation.md | ⚠️ | 0 | — |
| 08-management-governance/cloudwatch.md | ⬜ | 1 | — |
| 08-management-governance/systems-manager.md | ⚠️ | 0 | yes |
| 08-management-governance/x-ray.md | ⬜ | 0 | — |
| 09-migration-transfer/migration-services.md | ⬜ | 0 | — |
| 09-migration-transfer/snow-family.md | ⬜ | 0 | yes |
| 10-resilience-dr/aws-backup.md | ⚠️ | 0 | yes |
| 10-resilience-dr/disaster-recovery.md | ⚠️ | 0 | — |
| 10-resilience-dr/elastic-disaster-recovery.md | ⚠️ | 0 | yes |
| 11-other-services/hybrid-edge-infrastructure.md | ⚠️ | 0 | yes |
| 11-other-services/misc-managed-services.md | ⚠️ | 0 | — |
| 12-cost-optimization/cost-management-tools.md | ⬜ | 1 | yes |
| 13-machine-learning/ai-services.md | ⬜ | 3 | — |
| well-architected-framework.md | ⚠️ | 0 | yes |

## Priority Queue

Files confirmed still holding a raw-dump heading (⚠️), needing restructure into topic sections:

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

This file is a snapshot, not auto-maintained — re-run the grep below after each restructuring pass and update the table (flip ⚠️ → ✅, adjust cue counts):

```bash
for f in $(find . -name "*.md" -not -path "./README.md" -not -path "*/open-research-items.md" -not -path "*/exam-guide-mapping.md" -not -path "*/official-exam-guide.md" -not -path "*/content-review-status.md" | sort); do
  dump=$(grep -c -i "From slides (pages\|Answers (from slides" "$f" 2>/dev/null)
  cues=$(grep -c "Exam-wording cue" "$f" 2>/dev/null)
  research=$(grep -c "^## To research" "$f" 2>/dev/null)
  echo "$f | dump_headings=$dump | cues=$cues | to_research=$research"
done
```

⬜ rows are **not** verified done — they simply have no raw-dump heading. A file can still need restructuring/expansion in other ways (e.g. missing slide content, thin coverage) that this grep can't detect. Treat ⬜ as "not yet reviewed," not "confirmed fine."
