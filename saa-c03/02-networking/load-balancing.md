# Elastic Load Balancers

## Scalability & High Availability (prerequisite concepts)

Load balancing exists to serve these two goals — worth defining before ELB itself. Full detail (including EC2-specific vertical vs. horizontal scaling examples) lives in [ec2.md](../01-compute/ec2.md#scalability--high-availability).

- **Scalability** — a system's ability to handle greater load by adapting. Two kinds:
  - **Vertical Scalability** — make an individual instance bigger (scale up/down); has a hardware ceiling
  - **Horizontal Scalability** (= elasticity) — add more instances (scale out/in); implies a distributed system — this is what a load balancer distributes traffic across
- **High Availability** — running an application across at least 2 Availability Zones so it survives losing one; usually paired with horizontal scaling, but a distinct concept from it (can be passive, like an RDS Multi-AZ standby, or active, like a multi-AZ ASG behind a load balancer)
- A load balancer is the piece that turns a set of horizontally-scaled, multi-AZ instances into one available, seamless endpoint — hence why ELB, ASG, and Multi-AZ show up together constantly in architecture questions

## ELB Fundamentals

- A load balancer is a server that forwards traffic across multiple downstream instances/targets
- Why use one: spread load across instances, expose a single DNS entry point, handle instance failures seamlessly, run health checks, terminate SSL/TLS, support sticky sessions via cookies, separate public from private traffic
- Why use the *managed* ELB specifically over rolling your own: AWS guarantees it works, handles upgrades/maintenance/HA for you, and it's pre-integrated with EC2, ASG, ECS, ACM, CloudWatch, Route 53, WAF, and Global Accelerator — self-managing a load balancer is cheaper but far more operational effort
- Health checks run on a configured port + route (e.g. `/health`); any non-200 response marks the target unhealthy and stops traffic to it
- AWS has 4 generations: Classic (2009, legacy), Application (2016, Layer 7), Network (2017, Layer 4), Gateway (2020, Layer 3) — prefer the newer generations for new designs; each can be set up as internal (private) or external (public)

## Target Groups

The logical grouping of backend destinations a load balancer actually routes to — a layer of indirection between the LB's listener/routing rules and the real backends.

- Members can be: EC2 instances (by instance ID, optionally auto-registered/deregistered by an ASG), ECS tasks, IP addresses (private IPs — even ones outside the VPC, reachable via peering/VPN), Lambda functions (ALB only — the HTTP request is translated into a JSON event), or another Application Load Balancer (NLB target, for combining L4 + L7)
- Health checks are configured **per target group**, not per load balancer — protocol, port, path, thresholds, and interval all live here, and an unhealthy target is pulled from rotation automatically
- **One load balancer can point at multiple target groups** — this is how ALB path/host-based routing actually works under the hood (e.g. `/api/*` → target group A, `/images/*` → target group B), and each target group can use a different protocol/port than the others
- **Deregistration Delay** (the ALB/NLB equivalent of a CLB's Connection Draining) is set per target group — time to finish in-flight requests before pulling a de-registering/unhealthy target, 1-3600s, default 300s

## Sticky Sessions (Session Affinity)

- Supported on CLB, ALB, and NLB — routes a given client to the same target for the life of a cookie, instead of rebalancing every request
- Cookie source: an app-generated cookie, or a load-balancer-generated cookie (ALB uses `AWSALB`/`AWSALBAPP`)
- Trade-off: can unbalance load across backends, since "sticky" clients keep hitting the same target regardless of its current load

## Cross-Zone Load Balancing

Controls whether a load balancer node in one AZ can distribute requests to targets registered in a *different* AZ, instead of only targets in its own AZ.

- **ALB** — enabled by default; no charge for the inter-AZ data transfer
- **NLB / GWLB** — disabled by default; if you enable it, you pay for the inter-AZ data transfer
- **CLB** — disabled by default; no charge even if enabled

## SSL/TLS Certificates

- A load balancer uses an X.509 cert, managed via ACM or self-uploaded, to terminate SSL/TLS
- SNI (Server Name Indication) lets a single listener serve multiple certs/domains — works on ALB, NLB, and CloudFront, but **not** on CLB (which is why CLB needs one cert per hostname)

## Application Load Balancer (ALB)

- Layer 7 (HTTP/HTTPS) routing decisions
- Path-based, host-based, query-string, or header-based routing
- Can route to multiple ports per ECS container (dynamic port mapping)
- OIDC authentication via Cognito
- Health checks
- Access logs to S3
- Inserts the original client IP into the `X-Forwarded-For` request header (also `X-Forwarded-Port`, `X-Forwarded-Proto`), since the backend otherwise only sees the ALB's own IP

## Network Load Balancer (NLB)

- Layer 4 (TCP/SSL/UDP) routing
- Handles millions of requests/second at extremely low latency
- No path/host-based routing
- Health checks support the TCP, HTTP, and HTTPS protocols (not just TCP, despite NLB itself being Layer 4)
- Target group members: EC2 instances, IP addresses (must be private IPs), or an Application Load Balancer (lets you put an NLB in front of an ALB, e.g. for a static IP or PrivateLink)
- One static IP per AZ, with the option to attach an Elastic IP instead — useful for IP whitelisting downstream

## Classic Load Balancer (CLB)

- Supports TCP, SSL, HTTP, HTTPS
- Lower throughput/latency performance than NLB — legacy, avoid for new designs
- Supports only one SSL certificate — need multiple CLBs for multiple hostnames/certs
- "Connection Draining" is the CLB-specific name for what ALB/NLB call Deregistration Delay (see Target Groups above)

## Gateway Load Balancer (GWLB)

- Layer 3 (Network layer, IP packets) — deploys/scales/manages a fleet of 3rd-party virtual appliances (firewalls, IDS/IPS, deep packet inspection)
- Combines a transparent network gateway (single entry/exit point) with a load balancer
- Uses the GENEVE protocol on port 6081 — GENEVE (Generic Network Virtualization Encapsulation) wraps the original packet (any IP protocol) inside a UDP packet with an added metadata header, similar in purpose to VXLAN; this is what lets GWLB pass any traffic through to a 3rd-party appliance transparently for inspection, then send it back the same way
- Target group members: EC2 instances or private IP addresses (see Target Groups above)

## Notes

<!-- Your own notes go here. -->
