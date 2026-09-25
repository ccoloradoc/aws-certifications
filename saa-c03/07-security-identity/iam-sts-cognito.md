# IAM, STS & Cognito

## IAM (Identity and Access Management)

- **Permissions boundary** — sets the maximum permissions an IAM entity can have, regardless of attached policies; effective permissions are the **intersection** of the identity-based policy and the boundary — it never grants access on its own, only narrows what an identity-based policy would otherwise allow. Scoped to a single **user or role**, within one account; the standard use case is safely delegating IAM administration (e.g. letting a team lead create roles for their team without being able to grant those roles admin access)

> Exam-wording cue: "limit the maximum permissions a **specific IAM role/user** can have, even permissions delegated by someone else" → **Permissions Boundary**. "Limit the maximum permissions for an **entire AWS account or OU** in an Organization, affecting everyone including root" → **SCP** (Service Control Policy) — same "ceiling, not a grant" concept, but SCPs live in AWS Organizations and apply account/OU-wide, while a permissions boundary is IAM-level and applies to one identity.

- Global service; root account is created by default and shouldn't be used/shared day-to-day
- **Users** map to a physical person; **Groups** contain only users (not other groups); a user can belong to multiple groups or none
- **Policies** are JSON documents attached to users/groups defining permissions (apply least privilege)
  - Structure: `Version`, optional `Id`, one or more `Statement`s
  - Each statement: optional `Sid`, `Effect` (Allow/Deny), `Principal`, `Action`, `Resource`, optional `Condition`
- **Password policy**: minimum length, required character types, self-service password change, expiration, prevent re-use
- **MFA** = password + security device; protects the account if a password is stolen
  - Device options: virtual MFA app (supports multiple tokens per device), U2F security key (e.g. YubiKey), hardware key fob (Gemalto), GovCloud hardware key fob (SurePassID)
- Three ways to access AWS: Management Console (password + MFA), CLI (access keys), SDK (access keys)
  - Access Key ID ≈ username, Secret Access Key ≈ password — never share
- **IAM Roles** grant AWS services permission to act on your behalf (e.g. EC2 Instance Roles, Lambda Function Roles, CloudFormation Roles)
- Audit tools:
  - **IAM Credentials Report** (account-level) — lists all users and the status of their credentials
  - **IAM Access Advisor** (user-level) — shows granted permissions and last-accessed time, useful for trimming policies
- Best practices: don't use root except for account setup; lock or delete the root user's access keys; one physical user = one IAM user; assign permissions via groups, not individual users; enforce a strong password policy + MFA; use roles for AWS services; use access keys only for CLI/SDK, never hardcode/store them in code or an S3 bucket; audit regularly; never share IAM users or access keys

## AWS STS (Security Token Service)

- Issues temporary, limited-privilege credentials
- **Federation**: e.g., SAML 2.0 with Azure AD

### Single Sign-On Flow

1. Verify user against local IdP
2. Call `AssumeRole` or `GetFederationToken`
3. Request a sign-in token from the federation endpoint
4. Construct a console sign-in URL using that token

## Amazon Cognito

- Mobile/web app authentication
- **User Pools** — user directories with sign-up/sign-in
- **Identity Pools** — grant temporary AWS credentials, including for federated/SAML identities

### From slides (pages 421-570)

- Rule of thumb for Cognito vs. IAM: reach for Cognito when you have "hundreds/thousands of external users," "mobile users," or need to "authenticate with SAML"
- Cognito User Pools (CUP): serverless user directory; supports username/email+password login, password reset, email/phone verification, MFA, and federated logins (Facebook, Google, SAML); integrates directly with API Gateway and ALB for authentication
- Cognito Identity Pools (Federated Identities): exchange an identity (from a User Pool or a 3rd-party login) for temporary AWS credentials so users can call AWS services directly or through API Gateway; the IAM policy attached to those credentials is defined in Cognito and can be customized per `user_id` for fine-grained access (e.g. row-level security restricting a DynamoDB user to only their own items via `cognito-identity.amazonaws.com:sub` in the policy condition); default IAM roles exist for both authenticated and guest users

## Notes

<!-- Your own notes go here. -->

### From Netec live training (to review)

> Root user explicitly compared to Linux `root`: created automatically at signup, authenticated via the signup email+password, full unrestricted access to every service/resource — should never be used for day-to-day tasks, should be protected with MFA.
>
> — *Netec S1, 2:50:53-2:52:10*

> IAM framed as AWS's built-in identity provider, establishing the trust relationship that enables authentication and authorization — ties to "zero trust" (verify every request explicitly).
>
> — *Netec S1, 2:53:59-2:57:15*

> Principals defined as "AWS's name for anything that needs access to the account" — can be a user, a role, another service, or another identity provider.
>
> — *Netec S1, 2:57:30-3:00:17*

> IAM Users: an entity within an account, has its own credentials (password, access keys), has zero permissions by default — must be explicitly granted.
>
> — *Netec S1, 3:00:27-3:01:36*

> IAM users in practice are framed mostly for platform/console administration (admin/dev access, scripts/integrations) — complemented by IAM Identity Center for corporate/workforce identity.
>
> — *Netec S1, 3:02:05-3:03:37*

> Groups vs. Roles, framed live in Q&A: Groups = static membership/pertenencia (a user in a group inherits its permissions); Roles = flexible/temporary assignment of a function (e.g. a short-term project need or a night-shift task) with a definable lifecycle.
>
> — *Netec S1, 3:07:52-3:09:29*

> Roles have no username/password and no permanent access keys — that's why they're used for temporary access. Assumed via a permissions policy (what the role can do) + a trust policy (who can assume it); STS issues the temporary credentials once assumed.
>
> — *Netec S1, 3:07:57-3:14:17*

> Who/what can assume a role: an IAM user, another AWS service (Lambda → DynamoDB given as the example), an EC2 instance, a federated user, a principal in another AWS account (cross-account access), or an external identity provider (e.g. Active Directory).
>
> — *Netec S1, 2:57:50-2:58:31, 3:16:15-3:19:04*

> Programmatic access mechanics: `aws configure` via CLI needs an Access Key ID + Secret Access Key generated from the console; the CLI itself is multiplatform-installable.
>
> — *Netec S1, 3:06:18-3:06:54*

> Managed vs. custom policies: AWS ships pre-built/integrated policies for common job functions (e.g. "administrator access"); if that's too broad, create a custom policy instead.
>
> — *Netec S1, 3:07:26-3:08:00*
