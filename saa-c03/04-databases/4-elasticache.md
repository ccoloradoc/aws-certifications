# ElastiCache

- In-memory cache in front of RDS, Redshift, or S3 data
- Key/value store, OLAP-focused use cases
- Common use case: accelerating autocomplete/lookup queries
- Fully managed Redis or Memcached; AWS handles OS patching, setup, monitoring, failure recovery, backups — but adopting it does require app code changes

## Redis vs. Memcached

| | Redis | Memcached |
|---|---|---|
| Replication | Yes | No |
| High Availability | Yes | No |
| Security | Token/auth protection, in-transit encryption | Limited |
| Architecture | Single-threaded | Multi-core / multi-threaded |

- **Redis** additionally: Multi-AZ with auto-failover, read replicas for read scaling/HA, AOF-based durability, backup/restore, supports Sets/Sorted Sets (e.g. real-time gaming leaderboards)
- **Memcached** additionally: multi-node sharding (no replication/HA), non-persistent, backup/restore (serverless), multi-threaded

## Architecture Patterns

- **DB cache** — app queries cache first, falls back to RDS on miss and populates the cache; needs an invalidation strategy so stale data isn't served
- **User session store** — app writes session data to ElastiCache so any instance can retrieve it (keeps the app stateless)
  - **Exam scenario**: an app on an ASG + ALB keeps logging users out, and you don't want to enable Sticky Sessions because it risks overloading specific instances. Fix: store session data in ElastiCache (or DynamoDB) instead of on the instance — this makes the app stateless, so any instance can serve any user's request, the ALB can load-balance normally with no stickiness needed, and users stop losing their session

## Caching Patterns

- **Lazy Loading** — cache populated on read; can serve stale data until the next miss
- **Write-Through** — cache updated on every DB write; no stale data, but extra write latency
- **Session Store** — TTL-based temporary data (e.g. user sessions)

## Security

- IAM auth for Redis — note IAM policies only secure the AWS *API* layer (creating/managing the cluster), not the cache protocol itself
- Redis AUTH sets a password/token on the cluster as an extra layer on top of security groups
- Supports SSL in-flight encryption
- Memcached supports SASL-based authentication

## Notes

<!-- Your own notes go here. -->
