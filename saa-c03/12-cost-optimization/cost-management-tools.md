# Cost Management Tools

The base cheat sheet covered EC2 purchasing options but almost nothing else from the "cost-optimized architecture" exam domain (20% of the exam).

## To research

- **AWS Cost Explorer** — visualize and analyze spend over time
- **AWS Budgets** — set cost/usage thresholds and alerts
- **AWS Trusted Advisor** — automated checks across cost, performance, security, fault tolerance, service limits (note which checks require Business/Enterprise support)
- **AWS Compute Optimizer** — rightsizing recommendations for EC2, EBS, Lambda
- **Savings Plans** (Compute vs. EC2 Instance) vs. **Reserved Instances** — flexibility trade-offs (compare with [ec2.md](../01-compute/ec2.md) purchasing options)
- **Cost Allocation Tags** — attributing spend to teams/projects
- **AWS Pricing Calculator** — estimating costs before deployment
- Consolidated billing (see [organizations-control-tower.md](../07-security-identity/organizations-control-tower.md))

## Notes

<!-- Your own notes go here. -->

### Cost Explorer & Trusted Advisor detail (from slides, pages 721-870)

- **Cost Explorer**: visualize/analyze cost & usage over time; custom reports at account-wide or monthly/hourly/resource-level granularity; grouping by service, account, region, instance type, tag, API operation, AZ, etc.; recommends an optimal Savings Plan based on the last 60 days of usage (shows estimated $/hour commitment and projected monthly savings vs. On-Demand); forecasts usage up to 18 months out from historical trends
- **AWS Cost Anomaly Detection**: ML-based continuous monitoring that learns your normal spend pattern (no manual thresholds needed) to catch one-time spikes or sustained cost creep; scoped to services, member accounts, cost allocation tags, or cost categories; delivers a root-cause anomaly report via individual alerts or daily/weekly SNS summaries
- **Trusted Advisor**: agentless, high-level account assessment across Cost Optimization, Performance, Security, Fault Tolerance, Service Limits, and Operational Excellence; the full check set and programmatic access via the Support API require a Business or Enterprise support plan (the free tier only gets a limited check set)
