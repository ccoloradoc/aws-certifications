# Cost Management Tools

The base cheat sheet covered EC2 purchasing options but almost nothing else from the "cost-optimized architecture" exam domain (20% of the exam).

## To research

- **AWS Budgets** — set cost/usage thresholds and alerts
- **AWS Compute Optimizer** — rightsizing recommendations for EC2, EBS, Lambda
- **Savings Plans** (Compute vs. EC2 Instance) vs. **Reserved Instances** — flexibility trade-offs (compare with [ec2.md](../01-compute/ec2.md) purchasing options)
- **Cost Allocation Tags** — attributing spend to teams/projects
- **AWS Pricing Calculator** — estimating costs before deployment
- Consolidated billing (see [aws-organizations-and-control-tower.md](../07-security-identity/aws-organizations-and-control-tower.md))

## Cost Explorer

- Visualize/analyze cost & usage over time; custom reports at account-wide or monthly/hourly/resource-level granularity; grouping by service, account, region, instance type, tag, API operation, AZ, etc.
- Recommends an optimal Savings Plan based on the last 60 days of usage (shows estimated $/hour commitment and projected monthly savings vs. On-Demand)
- Forecasts usage up to 18 months out from historical trends

## AWS Cost Anomaly Detection

- ML-based continuous monitoring that learns your normal spend pattern (no manual thresholds needed) to catch one-time spikes or sustained cost creep
- Scoped to services, member accounts, cost allocation tags, or cost categories
- Delivers a root-cause anomaly report via individual alerts or daily/weekly SNS summaries

## AWS Trusted Advisor

- Agentless, high-level account assessment across Cost Optimization, Performance, Security, Fault Tolerance, Service Limits, and Operational Excellence
- The full check set and programmatic access via the Support API require a Business or Enterprise support plan (the free tier only gets a limited check set)

> Exam-wording cue: "visualize/analyze/forecast spend over time" → Cost Explorer. "Automatically flag an unusual spend spike with no threshold to configure" → Cost Anomaly Detection (contrast with Budgets, which needs a manual threshold you set yourself). "Broad best-practice check across cost *and* performance/security/fault-tolerance" → Trusted Advisor, not a cost-only tool despite living in this category.

## AWS Cost Optimization Hub

- Consolidates cost-optimization recommendations (rightsizing, idle-resource deletion, Reserved Instances, Savings Plans) across all accounts/Regions in one dashboard, accounting for your existing discounts/commercial terms — the aggregator sitting above tools like Compute Optimizer

## AWS Compute Optimizer

- Analyzes actual utilization metrics (EC2, EBS, Lambda) and recommends a better-fitting resource configuration (e.g. a different instance type) for performance, cost, or both

> Exam-wording cue: "**costs seem too high**," a modest multi-service footprint (EC2/RDS/S3), asked for a "**valid**" cost-optimization approach (not "the one best" answer) → **AWS Cost Optimization Hub** (consolidated, cross-account/Region view of idle resources, rightsizing, and RI/Savings Plan opportunities) **paired with AWS Compute Optimizer** (the detailed EC2 instance-type recommendation engine behind the rightsizing piece). Cost Optimization Hub aggregates *what* to look at; Compute Optimizer supplies the specific *instance-type* recommendation — the two work at different levels of the same problem, not as alternatives to each other.

## Notes

<!-- Your own notes go here. -->

Content sourced from the slide deck, pages 721-870, has been merged into the topical sections above (Cost Explorer, Cost Anomaly Detection, Trusted Advisor) rather than kept as a standalone slide-page dump.
