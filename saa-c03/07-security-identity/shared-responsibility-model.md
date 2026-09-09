# Shared Responsibility Model

Foundational security concept not covered in the base cheat sheet — expect this to be tested directly.

## To research

- "Security **of** the cloud" (AWS's responsibility) vs. "security **in** the cloud" (customer's responsibility)
- How responsibility shifts between IaaS (EC2), PaaS (RDS), and serverless (Lambda, DynamoDB) — customer manages less as AWS manages more
- Concrete examples: who patches the guest OS on EC2 vs. RDS vs. Lambda; who manages physical security; who configures security groups/IAM

## Notes

<!-- Your own notes go here. -->

### From Netec live training (to review)

> Using a cloud provider "transfers" physical/infrastructure security risk to AWS — AWS's own advanced physical controls and standards cover the data-center layer, no longer the customer's job. Explicitly tied back to the Shared Responsibility Model taught in Cloud Practitioner.
>
> — *Netec S1, 53:51-55:01, 1:24:19*
