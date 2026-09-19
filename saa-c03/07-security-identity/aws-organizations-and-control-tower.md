# AWS Organizations & Control Tower

The base cheat sheet mentioned SCPs (see [account-governance.md](account-governance.md)) but not Organizations itself.

## AWS Organizations

- Global service for managing multiple accounts: one **management account** + **member accounts**, each of which can only belong to one Organization
- **Consolidated Billing** — single payment method across all accounts, volume discounts on aggregated usage (e.g. EC2, S3), and shared Reserved Instance/Savings Plan discounts across accounts that wouldn't be reached individually
- API available to automate AWS account creation

## Organizational Units (OUs)

- OUs group accounts by purpose — typical patterns: business unit, environment lifecycle (dev/test/prod), or project-based
- Benefits of managing multiple accounts this way: consistent tagging standards for billing, org-wide CloudTrail sent to a central S3 account, org-wide CloudWatch Logs sent to a central logging account, and cross-account admin roles

## Service Control Policies (SCPs)

- Attach to OUs or accounts to restrict what users/roles in them can do — they never apply to the management account, which always retains full power
- Deny-by-default, just like IAM: access requires an explicit Allow all the way down from the root through every OU in the direct path to the target account
- Can be built as an **allowlist** or **blocklist** strategy
- **SCP vs. IAM Permissions Boundary**: an SCP is an org-wide/account-wide/OU-wide ceiling that can span multiple accounts; a Permissions Boundary is a per-user/per-role ceiling within a single account (not usable on groups) — good for letting one user self-manage policies without escalating to admin, or restricting one specific user rather than a whole account

## Tag Policies

- Standardize tag keys/allowed values org-wide; support Cost Allocation Tags and ABAC
- Block non-compliant tagging operations going forward (no effect on resources already tagged incorrectly)
- Generate compliance reports; can be monitored via EventBridge

## AWS Control Tower

- Builds a secure, compliant multi-account landing zone on top of Organizations in a few clicks; automates environment setup and ongoing policy management
- **Guardrails**: Preventive guardrails use SCPs (e.g. restrict regions across all accounts); Detective guardrails use AWS Config (e.g. flag untagged resources)
- Includes a compliance dashboard

> Exam-wording cue: Control Tower is not a replacement for Organizations — it's built *on top of* it. "Set up multi-account structure/SCPs manually" → Organizations. "Set up a compliant multi-account environment quickly, with automated guardrails and a dashboard, in a few clicks" → Control Tower.

## To research

- AWS Resource Access Manager (RAM) for sharing resources across accounts (see [migration-services.md](../09-migration-transfer/migration-services.md))

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 571-720.

### From Netec live training (to review)

> SCPs raised directly in response to a student comparing AWS to other clouds ("can I block a resource type/region from ever being created, like Azure Policy or Cloud Custodian?") — answer: yes, via Service Control Policies tied to AWS Organizations.
>
> — *Netec S1, 1:23:51-1:25:16, 3:26:20-3:27:36*

> SCP framed as operating at a higher hierarchical scale than identity/resource policies — org-wide governance (e.g. "don't allow work outside this one region") rather than per-user granularity.
>
> — *Netec S1, 3:26:36-3:27:11*

> AWS Organizations structure as a hierarchy: management/root account at the top → OUs group accounts by purpose (production/dev, department, security posture) → individual AWS accounts at the leaves, each an isolated workload environment. Policies applied at root cascade down; OUs can carry additional policies on top of what's inherited.
>
> — *Netec S2, 22:36-23:23, 23:23-27:44, 26:44-27:17*

> SCPs are the mechanism for maximum permission limits at the org level: block resource creation entirely (disallow a DB/instance type, restrict regions, deny specific services) to prevent accidental/unauthorized resource creation, control costs, enforce security — SCPs restrict what IAM is even allowed to permit, they don't grant permissions themselves.
>
> — *Netec S2, 27:44-28:43*

> Direct exam-tip quote: "if this is an exam question, they're very likely to ask about linking Service Control Policies to AWS Organizations" — the hierarchical relationship (SCP operates above account-level identity/resource policies) is explicitly called out as testable.
>
> — *Netec S2, 24:36-25:18*

> Consolidated billing named as a second major reason to use Organizations: a single bill across accounts, volume discounts (e.g. S3), shared Reserved Instance/Savings Plan discounts across accounts that wouldn't be reached individually.
>
> — *Netec S2, 29:24-30:30*

> AWS Control Tower: automates landing-zone setup on top of Organizations, sets a security baseline, automates account provisioning/governance for multi-account environments; combines preventive controls (via SCPs, e.g. blocking regions) with detective controls (via AWS Config, e.g. flagging configuration drift). Noted as out of scope for this specific cert but worth knowing conceptually.
>
> — *Netec S2, 39:06-41:55*

> Live Q&A: how do you tell if an account belonging to an org has a service blocked by an SCP? Answer: the error message returned when the blocked action is attempted generally indicates the block, for most resource types.
>
> — *Netec S2, 42:11-42:50*

> Concrete SCP-vs-identity-policy interaction example: an admin grants a user IAM permission for both EC2 and S3, but the org-level SCP only allows EC2 — result is an intersection: only the EC2 permission is effective, because the SCP acts as a ceiling regardless of what IAM grants underneath it.
>
> — *Netec S2, 33:38-34:57*

> Policy-evaluation chain reinforced (framed as "a chain/jerarquía"): SCP is checked first, then permission boundaries, then identity-based policies. Troubleshooting tip: if IAM says something should be allowed but it isn't working, check whether an org-level SCP is blocking it further up the chain.
>
> — *Netec S2, 36:47-38:17*
