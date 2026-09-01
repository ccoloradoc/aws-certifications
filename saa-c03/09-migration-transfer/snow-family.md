# AWS Snow Family

The base cheat sheet only mentioned Snowcone as a DataSync destination — the rest of the family wasn't covered.

## To research

- **AWS Snowcone** — smallest device, edge computing + data transfer
- **AWS Snowball Edge** (Storage Optimized / Compute Optimized) — larger-scale offline data transfer, can run compute (EC2/Lambda) at the edge
- **AWS Snowmobile** — exabyte-scale data transfer via shipping container
- When physical transfer beats network transfer (rule-of-thumb bandwidth/time calculations the exam likes to test)
- How Snow devices relate to DataSync and DMS (see [migration-services.md](migration-services.md))

## Answers (from slides, pages 271-420)

- Snowball Edge Storage Optimized: 104 vCPUs, 416GB RAM, 210TB SSD storage
- Snowball Edge Compute Optimized: 104 vCPUs, 416GB RAM, 28TB SSD storage — dedicated for edge compute use cases
- Edge computing use case: process data at locations with limited/no internet and no compute (a truck, a ship, a mining site) by running EC2 instances or Lambda functions directly on the Snowball Edge device — e.g. preprocessing, ML inference, media transcoding before shipping the device back
- Rule of thumb: if a transfer would take **more than a week** over your network, use Snowball instead. Rough transfer times: 10TB ≈ 12 days at 100Mbps / 30 hours at 1Gbps / 3 hours at 10Gbps; 100TB ≈ 124 days / 12 days / 30 hours; 1PB ≈ 3 years / 124 days / 12 days
- Snowball cannot import directly into Glacier — import to S3 first, then use an S3 Lifecycle policy to transition into Glacier

## Notes

<!-- Your own notes go here. -->
