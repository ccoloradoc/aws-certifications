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

### From Netec live training (to review)

> AWS Local Zones: extension of a parent Region into large population centers that lack a full Region; examples given: Los Angeles, Miami, New York (US), plus Mexico, Chile, Spain.
>
> — *Netec S1, 1:12:05-1:16:14*

> A Local Zone stays connected to, and is centrally administered from, its parent Region — explicitly not an independent Region.
>
> — *Netec S1, 1:13:45-1:14:21*

> Local Zone use cases: online gaming, real-time/live video editing, financial trading, ML inference — all latency-sensitive. Netflix cited by name as an adopter.
>
> — *Netec S1, 1:15:12-1:15:30*

> Services usable in a Local Zone named directly: EC2, EBS, VPC.
>
> — *Netec S1, 1:13:49-1:14:05*

> Rule-of-thumb distinction for the exam: Local Zone → compute/processing, Edge Location → content delivery/caching.
>
> — *Netec S1, 1:19:46-1:20:27*

> Edge/Perimeter locations described as "points of presence" distributed globally, used by CloudFront to cache content (images, video, APIs) closer to users, reducing latency vs. always hitting the origin region; supports Lambda@Edge.
>
> — *Netec S1, 1:16:23-1:19:16*
