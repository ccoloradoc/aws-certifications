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

### From Netec live training (to review)

> Session Manager named directly as the mechanism that let the lab's EC2 instance be administered "without leaving the console" (no separate SSH client).
>
> — *Netec S1, 2:45:32-2:45:49*

> Systems Manager introduced as a "Swiss Army knife" of operational tooling: instance connectivity, fleet management, patching — for both cloud and hybrid (on-prem) environments, via an SSM Agent installed on EC2 or on-prem servers.
>
> — *Netec S4, 1:13:08-1:13:55, 1:16:18-1:16:31*

> Run Command: execute a command/script remotely and at scale across many instances at once (10+, tag-based targeting) without going one-by-one — install software, restart a service, apply an update, without manually connecting to each host. Automation capability: e.g. automatically create an AMI from a failed instance for recovery, or automate OS/software updates for compliance.
>
> — *Netec S4, 1:14:35-1:17:03, 1:14:57-1:15:34*

> Auditing capability: integrates with session/activity logging for audit purposes; hybrid (on-prem + cloud) environments explicitly supported.
>
> — *Netec S4, 1:17:18-1:17:26, 1:17:26-1:17:41*

> Extended live exam-scenario debate on Patch Manager vs. Run Command, worth keeping as an illustration of exam nuance: scenario was EC2 Linux instances running third-party software needing a critical vulnerability remediated as fast as possible. Instructor initially endorsed Run Command over Patch Manager; a student pushed back that Patch Manager is typically understood as OS-level patching and may lack a predefined baseline for arbitrary third-party software, and that "as quickly as possible" favors immediate ad-hoc execution over Patch Manager's scheduled maintenance-window model. Counter-argument: Patch Manager can also apply immediately, not just on a schedule. Instructor's meta-conclusion: "we go back to the nature of certification exam questions — both answers can be correct, but one is more correct than the other; sometimes on the real exam there are questions that don't even carry a specific score."
>
> — *Netec S4, 1:24:28-1:27:39, 1:28:27-1:30:09, 1:31:58-1:32:15, 1:32:15-1:32:53*

> Second scenario, resolved clearly: instances need remote administration via a native AWS process, following Well-Architected best practices, with the least operational overhead — answer: Session Manager, with "native AWS service" flagged as the literal keyword in the stem pointing to the answer.
>
> — *Netec S4, 1:27:39-1:36:13, 1:36:13-1:36:40*
