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

## Notes

<!-- Your own notes go here. -->
