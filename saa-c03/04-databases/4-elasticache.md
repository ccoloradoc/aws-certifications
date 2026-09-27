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

> Exam-wording cue: "**robust disaster recovery strategy**" for an **ElastiCache Redis** caching layer, "**minimal downtime**," "**minimal data loss**," "**good application performance**" — with **no explicit mention of surviving a Region-level outage** → **Multi-AZ with automatic failover**. Redis Multi-AZ detects a primary node failure and promotes a replica in a different AZ within seconds, with the application still pointing at the **same endpoint** (no connection changes, no performance cost during normal operation) — satisfying all three requirements without any cross-region complexity. "Disaster recovery" in this kind of question is often used loosely to mean "recover automatically from a failure," not literal multi-Region DR — reach for **Global Datastore** instead only when the question explicitly names **surviving a Region outage** or **cross-region replication**, since that's a materially bigger (and unnecessary, if not asked for) feature to reach for.
- **Geospatial data type** (Redis only) — natively supports storing coordinates and querying by proximity (`GEOADD` to store a location, `GEOSEARCH`/`GEORADIUS` to find all points within a radius) — no manual geohashing or custom indexing needed, unlike a relational database or DynamoDB (which requires implementing your own geohash-based partition/sort key scheme for the same result)

> Exam-wording cue: "**real-time GPS coordinates**," "**match [X] with nearby [Y]**" (e.g. passengers with nearby drivers), an existing **RDS/read-replica setup** is **bottlenecked** by **thousands of updates/reads per second** for location data → **ElastiCache for Redis**, using its native **geospatial commands** (`GEOADD`/`GEOSEARCH`). This is a proximity-query workload, not a relational one — more read replicas only patches capacity on the wrong tool; Redis's in-memory speed plus built-in geospatial indexing is what actually matches both the "minimal latency" and "thousands of ops/sec" requirements. DynamoDB *can* do proximity matching too, but only via a manually-built geohash key scheme — Redis's native geo commands are the more direct fit when the question doesn't mention any such custom indexing already in place.

## Architecture Patterns

- **DB cache** — app queries cache first, falls back to RDS on miss and populates the cache; needs an invalidation strategy so stale data isn't served

> Exam-wording cue: "**frequent repeated queries**," a **read replica was already added but read traffic keeps spiking**, "**reduce repeated reads pressure**," "**cost-effective**" → **ElastiCache in front of Aurora/RDS**, using the **cache-aside pattern** (check cache → miss → query the database → write result back to cache). "Repeated" is the tell that the actual problem is redundant identical queries, not insufficient read capacity — adding more read replicas only lets you keep re-executing the same queries at growing cost, while a cache eliminates the redundant work entirely by serving repeat requests from memory. Distinct from **DAX**, which only applies in front of **DynamoDB** — for a relational database (RDS/Aurora), ElastiCache is always the caching answer, and it requires the application to implement the cache-check/cache-write logic itself (unlike DAX's transparent, API-compatible caching).
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
