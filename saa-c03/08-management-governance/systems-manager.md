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

## Notes

<!-- Your own notes go here. -->
