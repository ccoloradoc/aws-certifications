# AWS Certificate Manager (ACM)

Not mentioned in the base cheat sheet — ties directly into ALB and CloudFront setups already covered.

## To research

- Public vs. private certificates
- Free automatic renewal for ACM-issued public certificates
- Where certificates can be used (ALB, CloudFront, API Gateway) vs. where they can't (raw EC2 without a load balancer)
- Regional requirement: CloudFront requires the certificate to be requested in `us-east-1`

## Answers (from slides, pages 571-720)

- Public certificates are free; ACM auto-renews certificates it issued (starting 60 days before expiry) — but a certificate you generated elsewhere and *imported* into ACM has no auto-renewal (you must re-import before expiry; ACM sends expiration events starting 45 days out, and AWS Config has a managed rule `acm-certificate-expiration-check`)
- Requesting a public cert: list the FQDN(s) or a wildcard (`*.example.com`) → choose DNS validation (preferred, automatable via a Route 53 CNAME) or email validation (sent to WHOIS contacts) → takes a few hours to verify
- Usable on: ELB (CLB/ALB/NLB), CloudFront distributions, API Gateway custom domains — not directly on a raw EC2 instance without a load balancer/CloudFront in front
- Regional requirement confirmed: Edge-Optimized API Gateway (and CloudFront generally) needs the cert in `us-east-1`; Regional API Gateway needs the cert imported/created in the API's own region — either way, finish with a Route 53 CNAME or (better) A-Alias record

## Notes

<!-- Your own notes go here. -->
