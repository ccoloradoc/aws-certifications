# RDS Proxy

Not mentioned in the base cheat sheet — commonly tested in Lambda-to-RDS connection scenarios.

## What It Is

- Fully managed, serverless, auto-scaling, Multi-AZ (highly available) database proxy that sits in front of RDS/Aurora
- Requires no code changes for most applications

## Connection Pooling

- Pools and shares DB connections established by the app, reducing stress on the database (CPU/RAM) and minimizing open connections/timeouts — this is what solves connection exhaustion from many short-lived Lambda invocations connecting directly to the database
- **Exam phrasing to recognize**: "improves database efficiency by reducing the stress on database resources" is the exact benefit language AWS uses for RDS Proxy — if a question describes this outcome, RDS Proxy is very likely the intended answer

## Availability

- Reduces RDS/Aurora failover time by up to 66% compared to connecting directly
- **Exam phrasing to recognize**: "reduce failover time by up to 66%" is the exact benefit language AWS uses for RDS Proxy — a question quoting this number is very likely pointing at RDS Proxy

## Supported Engines

- **RDS**: MySQL, PostgreSQL, MariaDB, SQL Server
- **Aurora**: MySQL, PostgreSQL

## Security & Networking

- Enforces IAM Authentication and can pull DB credentials securely from AWS Secrets Manager
- **Exam gotcha**: RDS Proxy is a way to *enforce* IAM Authentication in front of engines/scenarios that wouldn't otherwise require it — if a question asks how to mandate IAM auth for database access, RDS Proxy is a valid answer, not just an incidental feature
- Never publicly accessible — must be accessed from within a VPC

## Notes

<!-- Your own notes go here. -->
