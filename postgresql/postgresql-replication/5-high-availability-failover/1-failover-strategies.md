---
source_course: "postgresql-replication"
source_lesson: "postgresql-replication-failover-strategies"
---

# Failover Strategies

## Introduction

A robust failover strategy is essential for production databases. PostgreSQL provides the building blocks for failover, but you need to architect the process, choose the right tools, and plan for every scenario.

This lesson covers planned and unplanned failover procedures, popular automation tools, and critical pre-failover verification steps.

## Key Concepts

- **Planned Failover (Switchover)**: A controlled transition from primary to standby, typically for maintenance or upgrades, with no data loss.
- **Unplanned Failover**: An emergency transition triggered by primary failure, where some data loss may occur depending on replication mode.
- **Split-Brain**: A dangerous condition where two servers both believe they are the primary, leading to data divergence.
- **Failover Automation**: Tools like Patroni, repmgr, and pg_auto_failover that detect failures and promote standbys automatically.

## Real World Context

A financial trading platform requires less than 30 seconds of downtime for any database failure. They deploy Patroni with etcd as the consensus store, running three PostgreSQL nodes across availability zones. When the primary becomes unreachable, Patroni detects the failure within seconds, verifies consensus among the remaining nodes, and promotes the most up-to-date standby automatically. The application connection pool detects the topology change and routes traffic to the new primary.

## Deep Dive

### Planned Failover (Switchover)

Used for maintenance, upgrades, or testing:

1. Stop writes to primary (set `default_transaction_read_only = on`)
2. Wait for standby to catch up (verify zero lag)
3. Promote standby
4. Reconfigure applications to connect to new primary
5. Reconfigure old primary as new standby

### Unplanned Failover

Used when the primary fails unexpectedly:

1. Detect primary failure
2. Verify standby is healthy
3. Promote the most up-to-date standby
4. Reconfigure applications
5. Investigate old primary

### Pre-Failover Verification

Before promoting a standby, verify it is ready:

```sql
-- On standby: Check replication lag before promotion
SELECT pg_wal_lsn_diff(
    pg_last_wal_receive_lsn(),
    pg_last_wal_replay_lsn()
) AS pending_bytes;

-- Check time since last transaction
SELECT now() - pg_last_xact_replay_timestamp() AS lag;

-- Promote only if lag is acceptable
SELECT pg_promote() WHERE (
    SELECT now() - pg_last_xact_replay_timestamp() < interval '5 seconds'
);
```

This conditional promotion prevents promoting a standby that is too far behind.

### Failover Automation Tools

Popular tools for automated failover:

| Tool | Type | Features |
|------|------|----------|
| Patroni | Cluster management | Automatic failover, etcd/ZooKeeper |
| repmgr | Replication manager | Manual/auto failover, monitoring |
| pg_auto_failover | Microsoft tool | Simple setup, automatic |
| Stolon | Kubernetes-native | Cloud-native, Consul/etcd |

Each tool has different strengths: Patroni is the most popular for its flexibility and strong community, repmgr is lightweight and well-established, pg_auto_failover is the simplest to set up, and Stolon is designed for Kubernetes environments.

## Common Pitfalls

- **Not testing failover regularly**: Failover procedures that are never tested will fail when you need them most. Schedule regular failover drills.
- **Promoting without checking lag**: Promoting a standby with significant lag means losing all transactions in the gap.
- **Split-brain after network partition**: Without a consensus mechanism (etcd, ZooKeeper), network partitions can lead to two primaries. Always use fencing or consensus-based tools.

## Best Practices

- Use a consensus-based tool like Patroni with etcd to prevent split-brain scenarios.
- Test failover monthly with planned switchovers and document the procedure.
- Automate pre-failover health checks to avoid promoting unhealthy standbys.

## Summary

- Planned failover (switchover) is controlled and lossless; unplanned failover responds to unexpected failures.
- Always verify standby lag before promotion to minimize data loss.
- Use automation tools like Patroni to detect failures and promote standbys within seconds.
- Consensus-based fencing prevents split-brain scenarios during network partitions.
- Regular failover testing is essential for production readiness.

## Code Examples

**Failover Health Checks**

```sql
-- Pre-failover checks on standby
SELECT
    pg_is_in_recovery() AS is_standby,
    pg_last_wal_receive_lsn() AS received,
    pg_last_wal_replay_lsn() AS replayed,
    pg_wal_lsn_diff(pg_last_wal_receive_lsn(), pg_last_wal_replay_lsn()) AS pending,
    now() - pg_last_xact_replay_timestamp() AS replay_lag;

-- Only promote if healthy
DO $$
BEGIN
    IF (SELECT now() - pg_last_xact_replay_timestamp()) < interval '10 seconds' THEN
        PERFORM pg_promote(true, 60);
        RAISE NOTICE 'Promotion initiated';
    ELSE
        RAISE EXCEPTION 'Standby too far behind for safe promotion';
    END IF;
END $$;
```


## Resources

- [Failover](https://www.postgresql.org/docs/current/warm-standby-failover.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*