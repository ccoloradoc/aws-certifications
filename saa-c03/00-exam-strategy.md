# Exam Strategy

## Answering SAA Questions

Exam scenarios present constraints and ask you to optimize for one of:

- Lowest cost
- Scalability
- Least effort / operational simplicity
- Use of managed services over self-managed

Identify which constraint the question is optimizing for before picking an answer — multiple options are often technically valid, but only one fits the stated constraint.

## Notes

<!-- Your own exam-taking notes, gotchas, and question patterns go here. -->

### From Netec live training (to review)

> Full worked scenario exercise (no single "correct" answer given): an on-prem e-learning platform (single physical server running web app + DB + media files, overwhelmed during peak training events) needs its compute layer moved to AWS to improve availability/scalability while optimizing operational cost. Proposed solutions ranged from a minimal EC2 Auto Scaling Group + ALB + Multi-AZ RDS design, up to Route 53 + CloudFront (media/global-latency angle) + API Gateway + EKS with HPA.
>
> — *Netec S2, 2:46:05-3:00:07*

> Instructor's takeaway: more services generally buys better availability/scalability but at higher operational cost/complexity — the "right" answer depends on which specific constraint the question stem emphasizes. A student pointed out that "optimize operational costs" refers to ongoing maintenance/operations effort, which is a different constraint than raw infrastructure spend — worth reading exam scenario wording carefully to know which cost dimension is meant.
>
> — *Netec S2, 2:54:51-2:58:19, 2:56:53-2:57:33*

> Explicit framing on exam question style: expect longer, more elaborate scenario-style questions (vs. shorter Foundational-cert wording) where you must extract the actual constraint being optimized for (cost, reduced operational load, a specific compliance/regulatory requirement, etc.) before picking an answer.
>
> — *Netec S3, 38:08-38:57*

> Four live exam-style storage practice scenarios: (1) a file server needing extended capacity with access-frequency-based tiering → Storage Gateway (File Gateway) + an S3 Lifecycle policy; (2) S3 records needing 1 year immediate access then 9 years archived → an S3 Lifecycle policy; (3) uploaded documents that can never be modified or deleted → S3 Object Lock, not "versioning + a read-only ACL" (an ACL can still be changed by an admin-permissioned user, so it doesn't guarantee true immutability) — a good illustration that the answer must satisfy the literal constraint in the stem, not just something plausible-sounding; (4) reducing data-transfer costs when sharing growing S3 data with a partner in another region → Requester Pays.
>
> — *Netec S3, 40:08-42:54, 43:38-44:52, 45:14-51:33, 51:56-56:18*

> General exam-answer heuristic repeated for emphasis: when a scenario references using a managed service instead of a raw instance, the expected answer in most cases is the managed service, because it reduces operational burden.
>
> — *Netec S3, 59:47-1:00:25*

> Worked multi-service migration reasoning (on-prem e-commerce MySQL DB, 500K+ users, performance/availability/scalability failures at peak): converged on splitting the workload by data shape rather than a single lift-and-shift — transactional/relational data → RDS/Aurora (Multi-AZ, automated backups, read replicas); unstructured/high-read data → DynamoDB (low latency, independent scaling); frequently-repeated reads → ElastiCache in front of both; DMS to handle the cutover from on-prem MySQL with minimal downtime. One answer also proposed Multi-AZ + cross-region replication on Aurora and converting DynamoDB tables to Global Tables for the DR angle.
>
> — *Netec S3, 1:59:07-2:18:15*

> General exam-technique commentary, given multiple times: long/extensive-seeming question stems often contain specific keywords ("native AWS service," "least operational overhead," "minimize cost," "supports interruptions," "minimal code changes") deliberately placed to point to the intended answer — read for those words rather than getting lost in the scenario detail. Elimination-by-descriptive-mismatch taught explicitly: cross out any option requiring unnecessary extra work the stem doesn't call for (building a custom AMI when none is needed, standing up a self-hosted registry when ECR already exists, refactoring code when the stem says "minimal changes").
>
> — *Netec S4, 1:35:14-1:36:40, 3:03:53-3:04:24, 1:37:59-1:39:20, 2:59:52-3:01:23*

> Direct acknowledgment that some real exam questions are genuinely ambiguous ("both answers can be correct, but one is more correct") and that a few questions on the actual exam reportedly don't even carry scoring weight — framed as a reason not to get stuck overthinking a single hard question. AWS's general exam-answer bias toward managed services over self-managed/DIY infrastructure restated multiple times as one of the most common filters.
>
> — *Netec S4, 1:32:15-1:32:53, 2:26:19-2:27:34, 2:56:09, 3:04:10*

> Worked scenario: migrating a .NET on-prem app backed by an Oracle database to AWS, requiring high availability with the least amount of change — converged on Elastic Beanstalk (.NET platform, Multi-AZ) for the app tier + RDS for the database (kept relational, avoided migrating to DynamoDB since that would require a schema/data-model rewrite and app-level library changes).
>
> — *Netec S4, 1:39:54-1:44:50*

> Worked scenario: a 3-tier app (web/app/DB layers on EC2) needs a more scalable architecture with the least possible change — converged on Elastic Beanstalk for the app/web tiers + RDS for the database + S3 for image/media storage (moving static assets out of the app layer specifically).
>
> — *Netec S4, 1:44:54-1:48:58*
