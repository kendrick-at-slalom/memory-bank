# PR #1308: Move session cache from Postgres to Redis

**Author:** @marcus
**Reviewers:** @priya, @platform-arch
**Status:** Open, awaiting architecture review

## What

This PR moves the session cache for `auth-service` from the Postgres-backed table we've been using to a Redis cluster. It violates ADR 0011 ("Postgres everywhere") and so is being filed as an Exception against that rule.

## Why

`auth-service` handles ~12k session lookups/sec at peak with a strict p99 budget of 8ms (set by the API gateway SLO). The Postgres-backed implementation has been hitting the budget about 60% of the time; the rest of the budget is consumed by GC pauses on connection pool spikes during peak.

We've spent two sprints tuning the Postgres path (connection pooling, prepared statements, table partitioning) and shaved a few ms off the median, but p99 is consistently over budget under load. Architecturally, session lookups are pure key-value reads with no relational shape; pushing them through a relational engine isn't earning anything for us.

## What changes

- New Redis cluster provisioned in the platform infrastructure account (`auth-session-cache`)
- `auth-service` reads/writes sessions via Redis client
- Postgres `sessions` table kept in place for now; will be deprecated once Redis is proven in production for 30 days
- Failover plan: if Redis is unreachable, fall back to Postgres. Adds latency in the failure mode, which is acceptable.

## Exception scope

This Exception covers `auth-service` specifically. It does not authorize Redis for any other service; future Redis use cases require separate Exceptions or a future amendment to ADR 0011.

## Compensating controls

- Redis cluster has the same backup, monitoring, and secret-rotation tooling as our Postgres clusters (we extended the platform tooling for this).
- Session loss on Redis failure is acceptable for this workload (sessions re-issue on next auth).
- Review date: 2026-10-15 (six months from merge).

## Approvers

- Architecture: @priya
- Platform: @platform-arch
- Owner of ADR 0011: @priya

## References

- ADR 0011 (Postgres everywhere)
- AUTH-204 (latency tracking ticket)
- Spike doc: `docs/spikes/auth-redis-spike.md` (in `auth-service` repo)
