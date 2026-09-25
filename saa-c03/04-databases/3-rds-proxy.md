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

## Combined with SQS: Resolving Throttling Under Bursty Traffic

A common combined scenario: Lambda connects directly to RDS/Aurora, and during a traffic spike Lambda scales out to hundreds/thousands of concurrent executions, each opening its own DB connection — RDS hits its connection limit and throws "too many connections" errors/timeouts. This isn't the DB being slow, it's too many *simultaneous connections* arriving at once, and it needs fixing on two fronts:

- **SQS** — put requests onto a queue instead of invoking Lambda directly/synchronously; this decouples producer from consumer, so Lambda polls the queue at a controlled rate (governed by batch size and **Reserved Concurrency**, which caps concurrent executions), smoothing a burst into a steady, bounded stream instead of it hitting the backend all at once
- **RDS Proxy** — pools/shares whatever connections those (now rate-limited) Lambda invocations do open, instead of each one opening a brand-new connection straight to the database

SQS reduces *how many* Lambda invocations hit the database concurrently; RDS Proxy makes *whatever connections do arrive* cheap and reusable instead of each being a brand-new connection.

> Exam-wording cue: "Lambda + RDS, connection errors/throttling during traffic **spikes**" → **SQS (decouple + rate-limit via Reserved Concurrency) + RDS Proxy (pool connections)**, both together. If the question describes timeouts/connection exhaustion under **steady** (not bursty) traffic, RDS Proxy alone is usually the intended answer, without needing SQS.

## Notes

<!-- Your own notes go here. -->
