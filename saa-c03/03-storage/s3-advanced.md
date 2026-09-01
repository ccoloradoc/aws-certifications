# S3 — Advanced Features

Follow-up to [s3.md](s3.md) — covers features the base cheat sheet didn't mention.

## To research

- **S3 Object Lock** — WORM (write-once-read-many) storage for compliance, governance vs. compliance mode
- **S3 Transfer Acceleration** — uses CloudFront edge locations to speed up uploads
- **Cross-Region Replication (CRR)** vs. **Same-Region Replication (SRR)**
- **S3 Batch Operations** — bulk operations on large numbers of objects
- Storage class deep dive: Glacier Instant/Flexible/Deep Archive retrieval tiers (see [glacier.md](glacier.md))
- Bucket policies vs. IAM policies vs. ACLs — when each applies
- S3 Event Notifications (to Lambda, SQS, SNS)

## Answers (from slides, pages 121-270)

- **S3 Object Lock** (requires versioning) — blocks version deletion for a set time
  - Compliance mode: nobody, including root, can overwrite/delete/shorten the retention
  - Governance mode: most users can't overwrite/delete, but specific users can be granted permission
  - Retention Period (fixed, extendable) is separate from Legal Hold (indefinite, freely added/removed via `s3:PutObjectLegalHold`)
- **S3 Transfer Acceleration** — routes uploads to the nearest AWS edge location, which forwards to the target-region bucket; compatible with multi-part upload
- **CRR vs SRR** — see the s3.md replication notes; both need versioning enabled on source and destination, are async, and can span accounts
- **S3 Batch Operations** — run one job (list of objects + action + params) across existing objects: modify metadata/tags/ACLs, copy between buckets, encrypt unencrypted objects, restore from Glacier, or invoke a Lambda per object; handles retries, progress tracking, completion notifications, and reports; pair with S3 Inventory + Athena to build the object list
- **Bucket policies vs IAM vs ACLs**: IAM policies are user-based (attached to an IAM identity); Bucket Policies are resource-based JSON (Effect/Principal/Action/Resource), set from the S3 console, and support cross-account access; Object/Bucket ACLs are older, finer/coarser-grained and can be disabled — an IAM principal can access an object if either the IAM policy OR the resource policy allows it, AND there's no explicit Deny; Block Public Access settings (account- or bucket-level) override ACL/policy-granted public access and exist specifically to prevent data leaks
- **S3 Event Notifications** — trigger on `ObjectCreated`, `ObjectRemoved`, `ObjectRestore`, `Replication`, etc., with optional filename filtering (e.g. `*.jpg`); typically deliver in seconds; can also route through **Amazon EventBridge** for JSON-rule filtering (by metadata/size/name) and multiple destinations (Step Functions, Kinesis Streams/Firehose) with archive/replay/reliable-delivery capabilities

### Additional S3 features (from slides, pages 121-270)

- Object encryption — 4 methods: SSE-S3 (AES-256, AWS-owned keys, default for new buckets/objects, header `x-amz-server-side-encryption: AES256`), SSE-KMS (customer-controlled KMS key with CloudTrail audit trail, header `aws:kms`, but upload/download calls count against KMS's per-second quota — request a quota increase if needed), SSE-C (customer-fully-managed key, S3 never stores it, HTTPS mandatory, key passed in every request header), and Client-Side Encryption (customer encrypts/decrypts before ever hitting S3)
- Encryption in transit = SSL/TLS via the HTTPS endpoint (vs. the unencrypted HTTP endpoint); can force it with a bucket policy checking `aws:SecureTransport: false` → Deny; bucket policies are evaluated before default (SSE-S3) encryption, so you can also force PUTs to include SSE-KMS/SSE-C headers
- CORS — Cross-Origin Resource Sharing controls whether a browser lets a request from one origin (scheme+host+port) reach another; configure `Access-Control-Allow-Origin` on the bucket to allow specific origins or `*`
- MFA Delete — requires versioning; required to permanently delete an object version or suspend versioning; NOT required to enable versioning or list deleted versions; only the bucket owner (root) can toggle it
- S3 Access Logs — logs every request (any account, authorized or denied) to a separate target bucket in the *same region*; never point the logging bucket at itself (creates an infinite logging loop)
- Pre-Signed URLs — generated via Console (1 min–12h), CLI (`--expires-in`, default 3600s, max 604800s/168h), or SDK; inherit the permissions of whoever generated them; useful for temporary GET/PUT access (premium downloads, dynamic user file lists, scoped upload locations)
- S3 Access Points — per-use-case named access to a bucket, each with its own DNS name and an access-point policy; a VPC-origin access point requires a VPC Endpoint (Gateway or Interface) whose policy also allows the bucket/access point
- S3 Object Lambda — runs a Lambda to transform an object on the way out (via an Object Lambda Access Point) without duplicating data; use cases: redacting PII, format conversion (XML→JSON), on-the-fly image resizing/watermarking
- Performance: S3 auto-scales to ~3,500 PUT/COPY/POST/DELETE or ~5,500 GET/HEAD requests/sec *per prefix*, with no cap on the number of prefixes — spreading reads across prefixes multiplies effective throughput; Byte-Range Fetches let you parallelize/resiliently retrieve partial objects
- S3 Storage Lens — org-wide storage analytics (30 days of usage/activity by default), aggregable by org/account/region/bucket/prefix, with default or custom dashboards, and optional daily metric export to S3 (CSV/Parquet); free tier gives ~28 metrics for 14 days, paid tier adds activity/cost/data-protection/status-code metrics, CloudWatch publishing, and prefix-level aggregation for 15 months

## Notes

<!-- Your own notes go here. -->
