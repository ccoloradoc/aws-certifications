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

### From Netec live training (to review)

> Caching framed generally: caching frequently-accessed data reduces direct database load, improves application performance, and can reduce cost. Lazy Loading walked through step by step: app checks cache first, on a miss queries the DB and writes the result back to cache — noted that cache storage is not unlimited, so a combination of strategies (including expiration) is recommended rather than relying on lazy loading alone. Write-through named as an alternative: the app writes to cache in parallel with the database rather than only populating on a miss.
>
> — *Netec S3, 1:39:58-1:40:36, 1:41:06-1:43:16, 1:43:25-1:44:13*

> TTL explained as the automatic-expiration mechanism needed regardless of caching pattern, since cache storage is finite — data is stored with a timestamp and evicted once TTL elapses; applies to both DynamoDB and ElastiCache.
>
> — *Netec S3, 1:44:20-1:45:50*

> ElastiCache named directly as the managed in-memory caching service — can accelerate response times from milliseconds down to microseconds, reduces DB query load, reduces cost by cutting direct DB hits. Redis named as an engine option with advanced capabilities (replication, HA, session management, advanced data structures). Memcached named as a simpler alternative engine for basic caching.
>
> — *Netec S3, 1:45:50-1:46:52, 1:46:58-1:47:36, 1:47:44-1:47:53*

> DynamoDB Accelerator (DAX) named directly as a caching layer purpose-built and fully integrated for DynamoDB specifically (distinct from general-purpose ElastiCache) — reduces DynamoDB read latency from milliseconds to microseconds by automatically caching reads; a query checks DAX first, falls back to DynamoDB on a miss and then populates the cache.
>
> — *Netec S3, 1:48:08-1:49:43*
