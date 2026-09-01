# IAM, STS & Cognito

## IAM (Identity and Access Management)

- **Permissions boundary** — sets the maximum permissions an IAM entity can have, regardless of attached policies
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
- Best practices: don't use root except for account setup; one physical user = one IAM user; assign permissions via groups, not individual users; enforce a strong password policy + MFA; use roles for AWS services; use access keys only for CLI/SDK; audit regularly; never share IAM users or access keys

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
