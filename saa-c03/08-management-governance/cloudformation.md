# AWS CloudFormation

- Text-based (JSON/YAML) infrastructure-as-code templates
- Maintains template version history
- **Update methods**: direct update, or **change sets** (preview changes before applying)
- **AWS SAM** (Serverless Application Model) — CloudFormation extension for serverless applications

## Notes

<!-- Your own notes go here. -->

### From slides (pages 721-870)

- Declarative: you describe the desired end state (resources + config) and CloudFormation figures out creation order/orchestration — no manual resource creation, all changes reviewed as code
- Cost visibility: every resource in a stack is tagged with a stack identifier, so per-stack cost is easy to track; a common dev-cost trick is auto-deleting/recreating dev stacks on a schedule (e.g. gone at 5pm, back at 8am)
- Supports nearly all AWS resources natively, plus **custom resources** for anything it doesn't
- **Infrastructure Composer** — visualizes a template's resources and their relationships as a diagram
- **CloudFormation Service Role** — an IAM role that lets CloudFormation create/update/delete a stack's resources on a user's behalf even if that user personally lacks permissions on those resources — supports least-privilege (grant stack-creation ability without granting the underlying resource permissions directly); requires the user to have `iam:PassRole`
