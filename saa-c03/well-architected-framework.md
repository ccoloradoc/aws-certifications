# AWS Well-Architected Framework

Cross-cutting framework that underlies how exam questions are worded ("optimize for cost", "improve reliability", etc.). Worth understanding before diving into individual services.

## Pillars (research each)

- Operational Excellence
- Security
- Reliability
- Performance Efficiency
- Cost Optimization
- Sustainability

## To research

- What each pillar's design principles are
- The AWS Well-Architected Tool (self-service review)
- How exam question wording maps back to a specific pillar

## Answers (from slides, pages 721-870)

- General guiding principles across all pillars: stop guessing capacity, test at production scale, automate to enable experimentation, allow for evolutionary architecture as requirements change, drive decisions with data, learn through "game days" (simulating flash-sale-style load)
- The 6 pillars are explicitly **not** trade-offs to balance against each other — they're meant to work in synergy
- **AWS Well-Architected Tool** — free; select a workload, answer a structured questionnaire, get your answers reviewed against the 6 pillars, and receive advice (docs/videos, a generated report, dashboard results)
- Related free tool: **AWS Trusted Advisor** (see [cost-management-tools.md](12-cost-optimization/cost-management-tools.md)) does an automated account-level check across similar categories (cost, performance, security, fault tolerance, service limits, operational excellence)

## Notes

<!-- Your own notes go here. -->

### From Netec live training (to review)

> The Well-Architected Framework exists so architects don't reinvent the wheel — a documented, structured checklist behind every "how do I build this correctly" question.
>
> — *Netec S1, 1:21:11-1:22:00*

> Pillars relevant to daily design questions: Security, Reliability, Performance Efficiency, Cost Optimization, Operational Excellence — cost optimization specifically called "always embedded," not an afterthought.
>
> — *Netec S1, 1:22:00-1:22:39, 56:22-57:20*

> The Framework isn't abstract — it directly maps to design questions like "should I protect the root account" or "should I use multiple AZs," each belonging to a specific pillar (security vs. reliability vs. performance).
>
> — *Netec S1, 1:29:42-1:30:11*

> Security pillar example content named directly: security design principles, traceability/monitoring, identity/infrastructure protection practices, threat detection.
>
> — *Netec S1, 1:30:42-1:31:13*

> AWS Well-Architected Tool workflow: select a workload → answer structured questions per pillar → save → get a report/recommendations; state is trackable over time to show improvement/drift.
>
> — *Netec S1, 1:29:06-1:35:16*

> AWS Well-Architected Labs: a separate, hands-on practice environment — reference architectures you implement yourself in your own account (not a course lab sandbox), tiered by difficulty per pillar.
>
> — *Netec S1, 1:35:41-1:36:53*
