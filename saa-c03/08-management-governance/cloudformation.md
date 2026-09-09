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

### From Netec live training (to review)

> Infrastructure-as-code framed via the problem it solves: manual console-driven resource creation is error-prone and doesn't reproduce cleanly across regions/environments — codifying infrastructure (like version-controlling application code) fixes both. Benefits: faster implementation, reduced human error, reusability (templates/modules), documentation, version-control history of infrastructure changes over time.
>
> — *Netec S4, 23:15-24:45, 24:56-26:05*

> Direct exam-tip quote: "if they ask about infrastructure as code in AWS, it's related to CloudFormation; when they mention a Stack, that's also CloudFormation." Mechanics: write a template (JSON/YAML) declaring resources → CloudFormation interprets it and makes the corresponding API calls → the result is a Stack.
>
> — *Netec S4, 35:03-35:40, 28:01-28:16, 28:32-30:06*

> Stack lifecycle operations: update, delete (removes resources consistently), detect configuration drift, and selectively preserve certain resources (e.g. keep a database's data) while changing others during a redeploy.
>
> — *Netec S4, 30:54-31:22*

> Use cases: deploying a full application architecture (LB, instances, RDS, S3) in parallel rather than manually clicking through each service; keeping dev/QA/production consistent with parameter-driven differences; integrating with CodePipeline/CodeBuild for CI/CD; enforcing required security configuration via template; quickly standing up/tearing down temporary test environments to control cost; deploying the same app across multiple regions/accounts for HA/DR.
>
> — *Netec S4, 31:40-34:25*

> Template structure: format/version, description, Parameters (dynamic values for reusability/customization), Conditions (e.g. only create a resource for a "production" environment), and the Resources section (the only genuinely required part). Outputs expose values (e.g. an instance ID) after a stack completes, for reference by other stacks/parameters.
>
> — *Netec S4, 37:40-39:52, 39:52-40:26*

> Layered architecture best practice: separate templates/stacks into logical layers so outputs from one feed the next — network (VPC, subnets, route tables, IGW, NAT GW), identity/security (IAM roles/policies/users/groups, least privilege), compute (instances, auto scaling — depends on network + identity), data (storage/databases), application (API Gateway, Lambda, etc.) — benefits: better organization/reuse, security isolation between environments, easier scaling.
>
> — *Netec S4, 41:07-45:14*

> Two authoring methods: raw JSON/YAML template, or build visually via CloudFormation Designer (drag-and-drop diagram + generated JSON side by side).
>
> — *Netec S4, 46:06-47:14*

> AWS Solutions Library named directly as an official AWS catalog of pre-built, architect-reviewed reference architectures, most shipping with a ready CloudFormation template, architecture explanation, step-by-step docs, and a cost estimate — explicitly aligned with the Well-Architected Framework. Example categories: static websites, image processing, serverless, security/monitoring/centralized-logging reference architectures, an AI/SageMaker example.
>
> — *Netec S4, 47:56-1:07:23*

> AWS CDK introduced as a layer on top of CloudFormation: instead of hand-writing JSON/YAML, you write actual programming code (Python, TypeScript, Java, etc.) that generates the CloudFormation template. Benefits: reusable code blocks/functions/classes for infrastructure, preferred by dev teams over learning YAML, useful for combining complex resources programmatically, fits naturally into a CI/CD pipeline.
>
> — *Netec S4, 1:08:07-1:09:38, 1:09:38-1:11:48*
