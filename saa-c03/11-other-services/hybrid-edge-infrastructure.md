# Hybrid & Edge Infrastructure

Not covered in the base cheat sheet — usually a small number of exam questions, but worth being able to distinguish these.

## To research

- **AWS Outposts** — AWS-managed hardware racks installed on-premises, extends AWS services into your own data center
- **AWS Local Zones** — low-latency AWS infrastructure closer to large population centers
- **AWS Wavelength** — AWS infrastructure embedded in telecom 5G networks for ultra-low-latency mobile use cases
- When each is the "correct" answer vs. Direct Connect (see [hybrid-connectivity.md](../02-networking/hybrid-connectivity.md)) or Transit Gateway

## Answers (from slides, pages 721-870)

- **AWS Outposts**: AWS-managed server racks installed in your own data center, giving the same services/APIs/tools on-prem as in the cloud — AWS handles setup and management of the rack itself, you're responsible for its physical security/power/space
  - Benefits: low-latency access to on-prem systems, local data processing, data residency, an easier on-prem→cloud migration path
  - Supported services include EC2, EBS, S3, EKS, ECS, RDS, EMR
  - This is the "correct answer" whenever a question needs AWS services running physically on-premises rather than merely connected to on-premises (that's Direct Connect/Transit Gateway territory instead)

## Notes

<!-- Your own notes go here. -->
