# AWS Organizations & Control Tower

The base cheat sheet mentioned SCPs (see [account-governance.md](account-governance.md)) but not Organizations itself.

## To research

- Organizational Units (OUs) and how SCPs attach to them
- Consolidated billing across member accounts
- AWS Control Tower — automated multi-account landing zone setup, guardrails (preventive vs. detective)
- Difference between an SCP and an IAM permissions boundary
- AWS Resource Access Manager (RAM) for sharing resources across accounts (see [migration-services.md](../09-migration-transfer/migration-services.md))

## Answers (from slides, pages 571-720)

- AWS Organizations is a global service for managing multiple accounts: one management account + member accounts (each can only belong to one Organization); gives Consolidated Billing (single payment method, volume discounts on EC2/S3 etc., shared RI/Savings Plan discounts across accounts); API available to automate account creation
- OUs can be organized by business unit, environment lifecycle, or project; typical benefits of multi-account-via-Organizations: consistent tagging for billing, org-wide CloudTrail into a central S3 account, org-wide CloudWatch Logs into a central logging account, cross-account admin roles
- **SCPs** attach to OUs or accounts to restrict what users/roles in them can do — they never apply to the management account (which always has full power); access requires an explicit Allow all the way down from the root through every OU in the path to the target account (deny-by-default, just like IAM); can be built as an allowlist or blocklist strategy
- **Tag Policies** — standardize tag keys/allowed values org-wide, support Cost Allocation Tags and ABAC, block non-compliant tagging operations (no effect on already-untagged resources), generate compliance reports, and can be monitored via EventBridge
- **SCP vs. IAM Permissions Boundary**: an SCP is an org-wide/account-wide/OU-wide ceiling on multiple accounts; a Permissions Boundary is a per-user/per-role ceiling within a single account (not usable on groups) — good for letting one user self-manage policies without being able to escalate to admin, or restricting one specific user rather than a whole account
- **AWS Control Tower**: builds a secure, compliant multi-account landing zone on top of Organizations in a few clicks; automates environment setup and ongoing policy management via **guardrails** — Preventive guardrails use SCPs (e.g. restrict regions across all accounts) and Detective guardrails use AWS Config (e.g. flag untagged resources); includes a compliance dashboard

## Notes

<!-- Your own notes go here. -->
