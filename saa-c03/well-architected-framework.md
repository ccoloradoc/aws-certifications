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
