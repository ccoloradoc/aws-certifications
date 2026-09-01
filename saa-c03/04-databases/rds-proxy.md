# RDS Proxy

Not mentioned in the base cheat sheet — commonly tested in Lambda-to-RDS connection scenarios.

## Answers (from slides, pages 121-270)

- Fully managed, serverless, auto-scaling, Multi-AZ (highly available) database proxy that sits in front of RDS/Aurora
- Pools and shares DB connections established by the app, reducing stress on the database (CPU/RAM) and minimizing open connections/timeouts — this is what solves connection exhaustion from many short-lived Lambda invocations connecting directly
- Reduces RDS/Aurora failover time by up to 66% vs. connecting directly
- Supports RDS engines (MySQL, PostgreSQL, MariaDB, SQL Server) and Aurora (MySQL, PostgreSQL)
- Enforces IAM Authentication and can pull DB credentials securely from AWS Secrets Manager
- Requires no code changes for most applications
- Never publicly accessible — must be accessed from within a VPC

## Notes

<!-- Your own notes go here. -->
