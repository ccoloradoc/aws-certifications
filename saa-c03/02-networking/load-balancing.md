# Elastic Load Balancers

## Application Load Balancer (ALB)

- Layer 7 (HTTP/HTTPS) routing decisions
- Path-based and host-based routing
- Can route to multiple ports per ECS container (dynamic port mapping)
- OIDC authentication via Cognito
- Health checks
- Access logs to S3

## Network Load Balancer (NLB)

- Layer 4 (TCP/SSL/UDP) routing
- Handles millions of requests/second at extremely low latency
- No path/host-based routing

## Classic Load Balancer (CLB)

- Supports TCP, SSL, HTTP, HTTPS
- Lower throughput/latency performance than NLB — legacy, avoid for new designs
- Supports only one SSL certificate — need multiple CLBs for multiple hostnames/certs

## Gateway Load Balancer (GWLB)

- Layer 3 (Network layer, IP packets) — deploys/scales/manages a fleet of 3rd-party virtual appliances (firewalls, IDS/IPS, deep packet inspection)
- Combines a transparent network gateway (single entry/exit point) with a load balancer
- Uses the GENEVE protocol on port 6081
- Target groups: EC2 instances or private IP addresses

## Notes

<!-- Your own notes go here. -->

### From slides (pages 121-270)

- Why use a load balancer: spread load across instances, single DNS entry point, handle instance failures seamlessly, health checks, SSL termination, sticky sessions via cookies, separate public/private traffic
- Health checks run on a port + route (e.g. `/health`); non-200 response marks the instance unhealthy
- Target groups: EC2 instances (can be ASG-managed), ECS tasks, Lambda functions (request translated to a JSON event), or private IP addresses — health checks are at the target-group level
- ALB routing: path-based, host-based, or based on query string/headers; great fit for microservices/containers; inserts client IP via `X-Forwarded-For` header (also `-Port`, `-Proto`)
- NLB has one static IP per AZ and supports an Elastic IP (useful for IP whitelisting)
- Sticky sessions (session affinity) — supported on CLB/ALB/NLB; app-generated cookie or ALB-generated `AWSALB`/`AWSALBAPP` cookie; can unbalance load across backends
- Cross-Zone Load Balancing: ALB enabled by default (no inter-AZ data charge); NLB/GWLB disabled by default (charged if enabled); CLB disabled by default (no charge if enabled)
- SSL/TLS: load balancer uses an X.509 cert, manageable via ACM or self-uploaded; SNI (Server Name Indication) lets one listener serve multiple certs/domains — works on ALB, NLB, CloudFront, but not CLB
- Connection Draining (CLB) / Deregistration Delay (ALB, NLB) — time to finish in-flight requests before deregistering an unhealthy/scaling-in instance; 1-3600s, default 300s, set low for short-lived requests
