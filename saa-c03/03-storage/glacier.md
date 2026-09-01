# S3 Glacier

- Slow retrieval by default (cold archival storage)
- **Expedited Retrieval** — 1–5 minutes, at extra cost

## Notes

<!-- Your own notes go here. -->

### From slides (pages 121-270)

- Pricing = storage cost + retrieval cost (unlike Standard/Intelligent-Tiering, which have no retrieval fee)
- **Glacier Instant Retrieval** — millisecond retrieval, good for data accessed ~once a quarter; 90-day minimum storage duration
- **Glacier Flexible Retrieval** (formerly "S3 Glacier") — Expedited (1-5 min), Standard (3-5h), Bulk (5-12h, free); 90-day minimum storage duration
- **Glacier Deep Archive** — for long-term storage: Standard (12h) or Bulk (48h) retrieval; 180-day minimum storage duration
- **S3 Glacier Vault Lock** — WORM model via a Vault Lock Policy that becomes immutable once locked; used for compliance/data retention
