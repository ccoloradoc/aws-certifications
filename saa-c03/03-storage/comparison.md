## Storage Comparison

| Service | Summary |
|---|---|
| [S3](s3.md) | Object storage |
| [S3 Glacier](glacier.md) | Object archival |
| [Instance Store](ebs-instance-store.md#instance-store) | Ephemeral block storage physically attached to the EC2 host |
| [EBS volumes](ebs-instance-store.md#ebs-elastic-block-store) | Network storage for one EC2 instance at a time |
| [EFS](efs.md) | Network file system for Linux instances, POSIX filesystem |
| [FSx for Windows](fsx.md#fsx-for-windows-file-server) | Network file system for Windows servers |
| [FSx for Lustre](fsx.md#fsx-for-lustre) | High-performance computing Linux file system |
| [FSx for NetApp ONTAP](fsx.md#fsx-for-netapp-ontap) | High OS compatibility |
| [FSx for OpenZFS](fsx.md#fsx-for-openzfs) | Managed ZFS file system |
| [Storage Gateway](storage-gateway.md) | S3 & FSx File Gateway, Volume Gateway (cache & stored), Tape Gateway |

## Relative Cost Ordering

The exam never tests exact $/GB figures (pricing changes too often), but the **relative ordering** by $/GB-month is stable and worth internalizing, since it tracks directly with each option's availability/durability/access-speed:

**EFS Standard** (most expensive) > **EBS gp3** > **S3 Standard** > **S3 Standard-IA** / **One Zone-IA** > **S3 Glacier Instant/Flexible Retrieval** > **S3 Glacier Deep Archive** (cheapest)

The pattern: fully-managed, multi-AZ, shared network filesystems (EFS) cost the most; single-AZ block storage (EBS) is next; general-purpose object storage (S3 Standard) is cheaper still; and cost keeps dropping as you trade away retrieval speed/immediacy for cold/archival tiers (down to Deep Archive at the bottom). A question framing a choice purely around "cheapest option that still meets the [durability/access-speed] requirement" is testing whether you know this ordering, not a specific number.

### Billing model gotcha: provisioned vs. consumed capacity

The $/GB ordering above assumes each service is billed for **actual data stored** — true for S3 and EFS, but **not** for EBS, which bills for the entire **provisioned volume size**, regardless of how much data actually lives on it.

**Classic exam scenario**: a small file (e.g. 1GB) is copied into S3 Standard, into a 100GB EBS (gp2) volume, and into EFS Standard. At the end of the month:
- **S3 Standard** and **EFS Standard** are billed for the actual **1GB** stored
- **EBS** is billed for the full **100GB provisioned**, even though only 1GB is used

**Correct order of charges, cheapest to most expensive: S3 Standard < EFS Standard < EBS** — despite EFS having a *higher* $/GB rate than EBS, EBS still ends up costing the most here, because it charges for capacity reserved, not data actually stored.

> Exam-wording cue: a scenario comparing storage costs for the *same small file* across S3/EFS/EBS is testing this exact distinction — **S3 and EFS bill only for actual data stored**; **EBS bills for the entire provisioned volume** — a small file on a large EBS volume can cost far more than the same file in S3 or EFS, even at a lower $/GB rate.