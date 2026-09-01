# ElastiCache

- In-memory cache in front of RDS, Redshift, or S3 data
- Key/value store, OLAP-focused use cases
- Common use case: accelerating autocomplete/lookup queries

## Redis vs. Memcached

| | Redis | Memcached |
|---|---|---|
| Replication | Yes | No |
| High Availability | Yes | No |
| Security | Token/auth protection, in-transit encryption | Limited |
| Architecture | Single-threaded | Multi-core / multi-threaded |

## Notes

<!-- Your own notes go here. -->

### From slides (pages 121-270)

- Fully managed Redis or Memcached; AWS handles OS patching, setup, monitoring, failure recovery, backups — but adopting it does require app code changes
- Architecture patterns:
  - DB cache — app queries cache first, falls back to RDS on miss and populates the cache; needs an invalidation strategy so stale data isn't served
  - User session store — app writes session data to ElastiCache so any instance can retrieve it (keeps the app stateless)
- Redis: Multi-AZ with auto-failover, read replicas for read scaling/HA, AOF-based durability, backup/restore, supports Sets/Sorted Sets (e.g. real-time gaming leaderboards)
- Memcached: multi-node sharding (no replication/HA), non-persistent, backup/restore (serverless), multi-threaded
- Security: IAM auth for Redis (IAM policies only secure the AWS API layer, not the cache protocol itself); Redis AUTH sets a password/token on the cluster as an extra layer on top of security groups; supports SSL in-flight encryption; Memcached supports SASL-based auth
- Caching patterns: Lazy Loading (cache on read, can go stale), Write-Through (cache updated on every DB write, no stale data), Session Store (TTL-based temporary data)
