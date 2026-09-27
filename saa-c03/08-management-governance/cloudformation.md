# AWS CloudFormation

- Text-based (JSON/YAML) infrastructure-as-code templates
- Maintains template version history
- **Update methods**: direct update, or **change sets** (preview changes before applying)
- **AWS SAM** (Serverless Application Model) — CloudFormation extension for serverless applications

## StackSets

- **AWS CloudFormation StackSets** — deploys **one template** across **many AWS accounts and Regions** in a single operation, centrally managed by an admin account; StackSets tracks and can re-apply the deployment as accounts/regions are added
- A **push** model: a central team enforces the same configuration everywhere, as opposed to **AWS Service Catalog**'s **self-service** model, where end users browse and launch from a catalog of pre-approved products

> Exam-wording cue: "**consistent resource provisioning** across **multiple AWS accounts/departments and Regions**," using the **same pre-defined configuration** (specific EC2 instance types, specific IAM roles, etc.) → **AWS CloudFormation StackSets** — deploys **one template** across many accounts/regions in a single operation, centrally enforced by an admin. This is a **push** model: distinct from **AWS Service Catalog**, which is a **self-service** model (end users browse and launch from a catalog of pre-approved products) — reach for Service Catalog when the question emphasizes end users **choosing** from approved options, and StackSets when it emphasizes a central team **enforcing the same configuration** everywhere.

## AWS Service Catalog

- Lets a central team define **"products"** — pre-approved CloudFormation templates (e.g. "an RDS instance with best-practice settings," "an EC2 instance of an approved type") — organized into **portfolios** that get shared with specific users/groups/accounts
- End users **self-service launch** only from that catalog, via the console or CLI — they never touch the underlying CloudFormation template or configure the resource manually themselves
- **Constraints** can restrict which parameters a launcher is even allowed to change (e.g. lock the instance type, or the encryption setting, to prevent deviation from the approved config)
- Distinct from **StackSets** — see the comparison cue above

> Exam-wording cue: "**mix of AWS experts and people learning AWS**," a user **misconfigured** a resource causing an outage, need **RDS/EC2/other best practices baked into a reusable template**, "**used by all your AWS users**" → **AWS Service Catalog** — publish a CloudFormation template encoding the correct configuration as a Service Catalog **product**, so every user self-service-provisions only from that vetted template instead of configuring the resource manually. A raw CloudFormation template alone doesn't solve this: nothing stops a user from still configuring the resource by hand in the console instead of using the template — Service Catalog is what makes the vetted template the **only** available self-service path.

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
