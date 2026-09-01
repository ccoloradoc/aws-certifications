# Hybrid Connectivity

## AWS Direct Connect (DX)

- Connects on-premises networks to one or multiple VPCs over a dedicated line
- Setup time: often > 1 month
- For resilience: add a 2nd DX connection, or use an IPSec VPN as backup

### Virtual Interfaces

- **Private VIF** — access to a VPC via private IP
- **Public VIF** — access to AWS public services via public IP
- **Transit VIF** — access to VPCs via Transit Gateway
- **Hosted VIF** — access shared across multiple accounts

### Data Transfer Tools

- **AWS DataSync** — copy data to S3, EFS, FSx, NFS, SMB, Snowcone
- **AWS DMS** (Database Migration Service) — copy/migrate databases
- **AWS SCT** (Schema Conversion Tool) — convert schemas between database engine types

## AWS Transit Gateway

- Central hub connecting on-premises networks and multiple VPCs
- Reduces operational complexity vs. mesh VPC peering
- Offers features beyond plain VPC peering (e.g., transitive routing)
- Access via a transit virtual interface
- Common pattern: **Direct Connect → Transit Gateway → Multiple VPCs**

### More Transit Gateway detail (from slides, pages 721-870)

- Regional resource, but Transit Gateways can be peered across regions; share cross-account via Resource Access Manager (RAM)
- Route tables on the TGW itself limit which attached VPCs can reach each other (segmentation)
- Works with Direct Connect Gateway and VPN connections; is the only AWS networking construct that supports IP Multicast
- **ECMP (Equal-Cost Multi-Path)** — spreads traffic across multiple Site-to-Site VPN tunnels to multiply bandwidth (e.g. combining tunnels for 2.5/5.0/7.5 Gbps); billed per-GB of TGW-processed data on top of the VPN cost
- Can also be used to share a single Direct Connect connection across multiple accounts

## AWS Site-to-Site VPN (from slides, pages 721-870)

- **Virtual Private Gateway (VGW)** — the AWS-side VPN concentrator, attached to the VPC you're connecting; ASN is customizable
- **Customer Gateway (CGW)** — the on-prem side (hardware or software); needs a public, internet-routable IP (or the public IP of a NAT-T-capable NAT device in front of it)
- Must enable **Route Propagation** on the VGW in the subnets' route table, and open ICMP inbound in security groups if you need to ping EC2 instances from on-prem
- **AWS VPN CloudHub** — low-cost hub-and-spoke model over multiple VPN connections terminating on the same VGW, for secure communication between multiple on-prem sites; still travels over the public internet; needs dynamic routing + route table config
- A Site-to-Site VPN is a common (cheaper) **backup** for a Direct Connect connection, instead of paying for a second DX connection

## AWS Direct Connect — additional detail (from slides, pages 721-870)

- Dedicated private connection from your DC to an AWS Direct Connect location; requires a VGW on your VPC; carries both public (S3) and private (EC2) traffic over the same connection
- Use cases: higher/cheaper bandwidth for large data sets, more consistent network performance for real-time feeds, hybrid environments; supports IPv4 and IPv6
- **Direct Connect Gateway** — needed when connecting one DX link to VPCs across multiple regions in the same account
- Connection types: Dedicated (1Gbps-400Gbps, a physical port requested via an AWS Direct Connect Partner) vs. Hosted (50Mbps-25Gbps, requested through a partner, capacity adjustable on demand) — either way, initial setup often takes over a month
- Data in transit over DX is private but **not encrypted**; pair DX with a Site-to-Site VPN for IPsec encryption at the cost of added complexity
- Resiliency: "High Resiliency" = one connection at multiple locations; "Maximum Resiliency" = separate connections terminating on separate devices at more than one location

## Notes

<!-- Your own notes go here. -->
