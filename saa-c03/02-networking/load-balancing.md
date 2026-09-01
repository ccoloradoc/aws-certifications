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

## Notes

<!-- Your own notes go here. -->
