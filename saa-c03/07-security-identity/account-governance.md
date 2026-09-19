# Account Governance

## Service Control Policies & IAM Policies

- **IAM policies** — restrict permissions within a single account
- **Service Control Policies (SCPs)** — restrict permissions across multiple accounts within an AWS Organization; see [aws-organizations-and-control-tower.md](aws-organizations-and-control-tower.md#service-control-policies-scps) for the full mechanics (deny-by-default evaluation, allowlist/blocklist strategies, SCP vs. IAM Permissions Boundary)
- Accounts can be migrated between AWS Organizations

> Exam-wording cue: SCPs never *grant* permissions on their own — they set the maximum boundary an account (and everything in it, including its root user) can ever do, evaluated on top of whatever IAM policies grant within that account. "Restrict what an entire account/OU can do" → SCP; "grant a user/role permission within one account" → IAM policy.

## IAM Policy Mechanics

- **IAM Roles vs. Resource-Based Policies** for cross-account access: assuming a role means giving up your own permissions for the role's; a resource-based policy (S3 bucket policy, SNS topic, SQS queue) lets the caller keep their own permissions while also being granted access to the resource — useful when, e.g., a user in Account A needs to read Account A's DynamoDB table *and* write to an S3 bucket in Account B without switching roles
- **IAM Policy Evaluation Logic**: evaluation starts assuming Deny; if any applicable policy has an explicit Deny, that wins immediately; otherwise SCPs, resource policies, and identity policies are all evaluated together — an explicit Allow somewhere along with no explicit Deny results in Allow
- IAM Conditions worth knowing: `aws:SourceIp` (restrict caller IP), `aws:RequestedRegion` (restrict target region), `ec2:ResourceTag`/`aws:PrincipalTag` (tag-based restrictions), `aws:MultiFactorAuthPresent` (require MFA for an action)
- S3 permission scope: actions like `s3:ListBucket` apply at the bucket level (`arn:...:bucket-name`); actions like `s3:GetObject`/`PutObject`/`DeleteObject` apply at the object level (`arn:...:bucket-name/*`)
- `aws:PrincipalOrgID` in a resource policy restricts access to any principal that's a member of a specific AWS Organization

## AWS IAM Identity Center

- Successor to AWS SSO — one login for AWS accounts in an Organization, SAML 2.0 business apps (Salesforce, Box, Microsoft 365), and EC2 Windows instances
- Identity sources: its own built-in identity store, or a 3rd party (Active Directory, OneLogin, Okta)
- Multi-account access via **Permission Sets** (bundles of IAM policies assigned to users/groups); **Application Assignments** give SSO into SAML apps; **ABAC** grants fine-grained permissions from user attributes (cost center, title, locale) stored in the Identity Store, so access changes just by editing attributes
- AWS Directory Service options for on-prem AD integration: **AWS Managed Microsoft AD** (your own AD in AWS, can trust on-prem AD), **AD Connector** (proxy redirecting to on-prem AD, users stay managed there), **Simple AD** (AD-compatible but cannot join an on-prem AD)

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 571-720.

### From Netec live training (to review)

> Policy evaluation walkthrough: check permission-boundary/SCP filter first → explicit Deny anywhere wins outright → explicit Allow (with no Deny) results in Allow → otherwise implicit deny by default — explicitly compared to how a firewall default-denies.
>
> — *Netec S1, 3:38:05-3:39:12*

> Live Q&A on that mechanic: a student asked what happens when a user is in two groups with conflicting Lambda permissions; answer reinforced that a permission-boundary/SCP-level deny is checked first and wins regardless of what identity-based policies grant.
>
> — *Netec S1, 3:24:02-3:24:34*

> Identity-based vs. resource-based policy, demoed on an S3 bucket: the bucket policy is resource-based and must name the Principal explicitly (who gets access); an identity-based policy (on the user) must instead name the Resource — structural mirror images of each other.
>
> — *Netec S1, 3:31:03-3:32:00, 3:39:33-3:39:59*

> Defense-in-depth via IAM layering: even if a user's identity policy grants S3 upload, a bucket policy can independently block it (or vice versa) — "one element doesn't replace another, it's a chain."
>
> — *Netec S1, 3:28:17-3:29:02, 3:30:28-3:31:31*

> Separate defense-in-depth example (non-IAM): storing user/admin passwords in plaintext vs. hashed/encrypted — a SQL-injection breach is far more damaging in the plaintext case; illustrates that defense-in-depth mitigates blast radius rather than preventing the initial incident.
>
> — *Netec S1, 3:40:30-3:41:20*

> Policy JSON structure walked through directly: `Effect` (Allow/Deny), `Action` (e.g. list/get on S3), `Resource` (specific bucket/object ARN or `*` wildcard), optional `Condition` (e.g. resource tag match).
>
> — *Netec S1, 3:34:01-3:35:23*

> Explicit warning: watch for wildcard (`*`) actions/resources granting more than intended — a common exam trap and a real-world security risk.
>
> — *Netec S1, 3:34:01-3:34:17*

> Granularity gotcha: permission on the bucket ARN doesn't automatically cover the bucket's object ARNs, and vice versa — flagged as a plausible exam question.
>
> — *Netec S1, 3:36:15-3:36:46*

> Same SCP → permission-boundary → identity-policy evaluation order reinforced with a second walkthrough in session 2 — see the equivalent blockquotes in [aws-organizations-and-control-tower.md](aws-organizations-and-control-tower.md).
>
> — *Netec S2, 29:07-29:23, 35:14-38:23*
