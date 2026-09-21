# AWS Certificate Manager (ACM)

Not mentioned in the base cheat sheet — ties directly into ALB and CloudFront setups already covered.

## Overview

- Easily provision, manage, and deploy TLS certificates
- Provides in-flight encryption for websites (HTTPS)
- Supports both public and private TLS certificates
- Public TLS certificates are free of charge
- Automatic TLS certificate renewal
- Integrates with (you can load certificates on): Elastic Load Balancers (CLB, ALB, NLB), CloudFront distributions, and APIs on API Gateway
- Not directly usable on a raw EC2 instance without a load balancer/CloudFront in front

## Requesting Public Certificates

1. List the domain names to include: a Fully Qualified Domain Name (`corp.example.com`) or a wildcard (`*.example.com`)
2. Select a validation method:
   - **DNS validation** — preferred for automation; adds a CNAME record to your DNS config (e.g. Route 53)
   - **Email validation** — sends emails to the contact addresses in the WHOIS database
3. Verification takes a few hours
4. The public certificate is enrolled for automatic renewal — ACM renews ACM-generated certificates **60 days before expiry**

## Importing Certificates

- Option to generate the certificate outside ACM and import it
- **No automatic renewal** — you must import a new certificate before expiry
- ACM sends daily expiration events starting **45 days** before expiry (the number of days is configurable); the events appear in EventBridge
- AWS Config has a managed rule, `acm-certificate-expiration-check`, that checks for expiring certificates (configurable number of days) — see [aws-config.md](aws-config.md)

## Integrations

### Load Balancers

- The load balancer uses an X.509 certificate (SSL/TLS server certificate) on its HTTPS listener
- You can manage certificates with ACM, or alternatively upload your own — see [load-balancing.md](../02-networking/load-balancing.md)

### API Gateway

- Create a **Custom Domain Name** in API Gateway to get HTTPS on it, with the certificate from ACM
- **Edge-Optimized** (default; for global clients; requests are routed through CloudFront edge locations, though the API still lives in one Region) — the certificate must be in **`us-east-1`**, the same Region as CloudFront
- **Regional** (for clients in the same Region) — the certificate must be imported on API Gateway, in the same Region as the API stage
- **Private** endpoints (reachable only from your VPC via an interface VPC endpoint) — the slides don't cover a certificate for this type
- Either way, finish by setting up a Route 53 **CNAME** or (better) **A-Alias** record

### CloudFront

- The certificate must be requested in `us-east-1` (same rule as Edge-Optimized API Gateway) — see [cloudfront-and-global-accelerator.md](../02-networking/cloudfront-and-global-accelerator.md)

> Exam-wording cue: "HTTPS on a CloudFront distribution or an Edge-Optimized API Gateway custom domain" → request/import the certificate in **`us-east-1`**; "Regional API Gateway" → certificate in the **API's own Region**. "Certificate is about to expire and wasn't renewed automatically" → it was an **imported** certificate (ACM only auto-renews the ones it issued); "get alerted before it expires" → EventBridge expiration events or the Config rule `acm-certificate-expiration-check`.

## Private Certificates (AWS Private CA)

- **AWS Private Certificate Authority (Private CA)** — managed service to create your own private CA in your account and manage the lifecycle of the private certificates it issues
- A CA is created as a **root** or **subordinate** CA
- Certificates are issued from a specific CA, e.g. by submitting a **CSR** with a signing algorithm and a validity period (API: `acm-pca issue-certificate`)
- Private certificates are for **internal** resources (private hostnames, internal services) — clients must be configured to trust your private CA; they aren't publicly trusted like ACM public certificates
- The slides only state that ACM supports private certificates; the details in this section come from the AWS docs/CLI reference, not the deck. Pricing and auto-renewal behavior for private certificates weren't confirmed

> Exam-wording cue: "TLS for internal/private domain names, not reachable from the internet" or "run your own CA without managing CA infrastructure" → **ACM Private CA**; "public website HTTPS, free" → ACM **public** certificate.

## Notes

<!-- Your own notes go here. -->

Private CA section sourced from the AWS docs, not the slide deck. Other content sourced from slide deck, pages 661-690 (ACM overview, requesting/importing certificates, API Gateway integration), with the ALB and API Gateway endpoint detail from pages 121-150 and 481-510.
