# Account Governance

- **Service Control Policies (SCPs)** — restrict permissions across multiple accounts within an AWS Organization
- **IAM policies** — restrict permissions within a single account
- Accounts can be migrated between AWS Organizations

## Notes

<!-- Your own notes go here. -->

### IAM policy mechanics (from slides, pages 571-720)

- **IAM Roles vs. Resource-Based Policies** for cross-account access: assuming a role means giving up your own permissions for the role's; a resource-based policy (S3 bucket policy, SNS topic, SQS queue) lets the caller keep their own permissions while also being granted access to the resource — useful when, e.g., a user in Account A needs to read Account A's DynamoDB table *and* write to an S3 bucket in Account B without switching roles
- **IAM Policy Evaluation Logic**: evaluation starts assuming Deny; if any applicable policy has an explicit Deny, that wins immediately; otherwise SCPs, resource policies, and identity policies are all evaluated together — an explicit Allow somewhere along with no explicit Deny results in Allow
- IAM Conditions worth knowing: `aws:SourceIp` (restrict caller IP), `aws:RequestedRegion` (restrict target region), `ec2:ResourceTag`/`aws:PrincipalTag` (tag-based restrictions), `aws:MultiFactorAuthPresent` (require MFA for an action)
- S3 permission scope: actions like `s3:ListBucket` apply at the bucket level (`arn:...:bucket-name`); actions like `s3:GetObject`/`PutObject`/`DeleteObject` apply at the object level (`arn:...:bucket-name/*`)
- `aws:PrincipalOrgID` in a resource policy restricts access to any principal that's a member of a specific AWS Organization

### AWS IAM Identity Center (from slides, pages 571-720)

- Successor to AWS SSO — one login for AWS accounts in an Organization, SAML 2.0 business apps (Salesforce, Box, Microsoft 365), and EC2 Windows instances
- Identity sources: its own built-in identity store, or a 3rd party (Active Directory, OneLogin, Okta)
- Multi-account access via **Permission Sets** (bundles of IAM policies assigned to users/groups); **Application Assignments** give SSO into SAML apps; **ABAC** grants fine-grained permissions from user attributes (cost center, title, locale) stored in the Identity Store, so access changes just by editing attributes
- AWS Directory Service options for on-prem AD integration: **AWS Managed Microsoft AD** (your own AD in AWS, can trust on-prem AD), **AD Connector** (proxy redirecting to on-prem AD, users stay managed there), **Simple AD** (AD-compatible but cannot join an on-prem AD)
