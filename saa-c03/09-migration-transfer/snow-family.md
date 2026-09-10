# AWS Snow Family

The base cheat sheet only mentioned Snowcone as a DataSync destination — the rest of the family wasn't covered.

## To research

- **AWS Snowcone** — smallest device, edge computing + data transfer (specs/use case detail)
- **AWS Snowmobile** — exabyte-scale data transfer via shipping container (deck only positions it alongside Snowball as a PB-scale mover, no device-level specifics)

## Overview

- Highly-secure, portable devices to collect and process data at the edge, and migrate data into and out of AWS — helps migrate up to petabytes of data
- Family: **Snowcone** (smallest), **Snowball Edge** (Storage Optimized / Compute Optimized), **Snowmobile** (largest, exabyte-scale)

## Device Specs

- **Snowball Edge Storage Optimized**: 104 vCPUs, 416GB RAM, 210TB SSD storage
- **Snowball Edge Compute Optimized**: 104 vCPUs, 416GB RAM, 28TB SSD storage — dedicated for edge compute use cases

## Edge Computing

- Use case: process data at locations with limited/no internet and no compute (a truck, a ship, a mining site) by running EC2 instances or Lambda functions directly on the Snowball Edge device
- Both Compute Optimized (dedicated for this) and Storage Optimized support it
- Examples: preprocessing data, machine learning inference, transcoding media — before shipping the device back

## Snowball vs. Network Transfer

- Challenges with network-based migration: limited connectivity, limited bandwidth, high network cost, shared bandwidth (can't maximize the line), connection stability
- Rule of thumb: if a transfer would take **more than a week** over your network, use Snowball instead
- Rough transfer times: 10TB ≈ 12 days at 100Mbps / 30 hours at 1Gbps / 3 hours at 10Gbps; 100TB ≈ 124 days / 12 days / 30 hours; 1PB ≈ 3 years / 124 days / 12 days
- Worked example: 200TB over a 100Mbps internet/VPN link ≈ 185 days; over a 1Gbps Direct Connect (see [hybrid-connectivity.md](../02-networking/hybrid-connectivity.md)) ≈ 18.5 days, but DX's one-time setup itself often takes a month+; via Snowball ≈ about a week end-to-end

## Snow Family in Migration Architectures

- Positioning alongside other transfer services: Direct Connect moves GB/s over a private secure network; Snowball & Snowmobile physically move PB-scale data; DataSync schedules ongoing syncs between on-premises and S3/EFS/FSx
- A Snowball transfer can be combined with **AWS DMS** for the cutover step; for ongoing replication/transfers, prefer Site-to-Site VPN or Direct Connect paired with DMS or DataSync rather than repeated one-off physical transfers
- See [migration-services.md](migration-services.md) for the full DataSync/DMS/SCT comparison

## Snowball → Glacier

> **Q:** You want to migrate a large on-premises dataset directly into Amazon S3 Glacier using a Snowball device. Can you import straight into Glacier?
>
> **A:** No — Snowball cannot import directly into Glacier. Import into S3 first, then use an S3 Lifecycle policy to transition the data into Glacier (see [glacier.md](../03-storage/glacier.md)).

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 331-360, 361-390, 781-810, and 811-840.

### From Netec live training (to review)

> Snow Family framed as the offline transfer option — physical devices shipped by AWS, used to copy data locally then physically returned to AWS for upload; used when network transfer isn't fast/available enough for the data volume.
>
> — *Netec S3, 35:51-36:37*

> AWS Snowcone capacity confirmed directly as ~8TB. Snowball named (without further specs) as the option for larger migrations.
>
> — *Netec S3, 36:46-36:54, 36:54-37:12*
