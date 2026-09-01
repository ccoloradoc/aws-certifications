# IAM, STS & Cognito

## IAM (Identity and Access Management)

- **Permissions boundary** — sets the maximum permissions an IAM entity can have, regardless of attached policies

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

## Notes

<!-- Your own notes go here. -->
