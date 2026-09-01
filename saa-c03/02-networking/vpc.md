# VPC

## Core Concepts

- VPCs map to Regions; subnets map 1-to-1 with Availability Zones
- Route tables determine traffic direction
- Internet Gateways connect VPCs to the internet
- VPC Endpoints provide private connections to AWS services
- VPC Peering routes traffic directly between two VPCs
- Transit Gateway acts as a central hub for many VPCs (see [hybrid-connectivity.md](hybrid-connectivity.md))

## Subnet Types

### Public Subnets

- Route to an Internet Gateway
- Auto-assign public IP/ENI to EC2 instances
- Allow SSH (port 22) for administration

### Private Subnets

- Route outbound traffic through NAT devices
- **NAT Gateway** (AWS-managed) vs. **NAT Instance** (manually managed, can double as bastion host)
- Use a bastion host for SSH access into private subnets

## VPC Endpoints

### Interface Endpoints

- Private connections via AWS PrivateLink
- Apply to many services (API Gateway, CloudFormation, CloudWatch, S3, etc.)
- Backed by an ENI with a private IP, traffic directed via DNS

### Gateway Endpoints

- Direct private-IP access to S3 or DynamoDB only
- Traffic managed via route tables
- Protected by VPC endpoint policies

## Network Adapters

- **ENI** — basic elastic network interface
- **ENA** — enhanced networking adapter
- **EFA** — elastic fabric adapter, for HPC

## Security Groups vs. Network ACLs

| Feature | Security Group | NACL |
|---|---|---|
| Level | Instance | Subnet |
| Rules | Allow only; stateful | Allow/Deny; stateless |
| Evaluation | All rules together | In order |
| Default inbound | Deny (custom); Allow (default) | Deny |
| Default outbound | Allow all | Allow all (default VPC) |

## Notes

<!-- Your own notes go here. -->
