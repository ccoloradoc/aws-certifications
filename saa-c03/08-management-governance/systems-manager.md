# AWS Systems Manager

The base cheat sheet only mentioned "Run Command" — the rest of the SSM suite is untouched.

## To research

- **Parameter Store** — configuration/secrets storage (see [secrets-management.md](../07-security-identity/secrets-management.md) for comparison with Secrets Manager)
- **Session Manager** — shell access to instances without SSH/bastion hosts (contrast with the bastion host pattern in [vpc.md](../02-networking/vpc.md))
- **Patch Manager** — automated OS/software patching
- **Run Command** — execute commands across instances without SSH (already noted in [ec2.md](../01-compute/ec2.md))
- **State Manager** — maintain a defined configuration state on instances
- **Automation** — runbooks for common maintenance/deployment tasks
- Requires the SSM Agent running on managed instances

## Answers (from slides, pages 721-870)

- **Session Manager** — secure shell into EC2/on-prem servers with no SSH, bastion host, SSH keys, or port 22 needed; supports Linux/macOS/Windows; session logs can go to S3 or CloudWatch Logs
- **Run Command** — run a document (script) or ad-hoc command across many instances at once (via resource groups), no SSH required; output visible in-console or sent to S3/CloudWatch Logs; can notify SNS on status; integrated with IAM/CloudTrail; invokable from EventBridge
- **Patch Manager** — automates OS/app/security patching for EC2 and on-prem servers (Linux/macOS/Windows), on-demand or on a schedule via Maintenance Windows; generates a patch-compliance report of what's missing
- **Maintenance Windows** — a defined schedule (window + duration + registered instances) for running disruptive tasks like patching or driver updates
- **Automation** — runs "Automation Runbooks" (SSM Documents, pre-built or custom) for common maintenance tasks (restart an instance, create an AMI/EBS snapshot); triggerable manually, via EventBridge, on a Maintenance Window schedule, or by AWS Config as a remediation action

## Notes

<!-- Your own notes go here. -->
