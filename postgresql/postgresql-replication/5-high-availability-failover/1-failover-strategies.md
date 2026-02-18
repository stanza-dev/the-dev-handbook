---
source_course: "postgresql-replication"
source_lesson: "failover-strategies"
---

# Failover Strategies

A robust failover strategy is essential for production databases. PostgreSQL provides the building blocks, but you need to architect the failover process.

## Failover Types

### Planned Failover (Switchover)

Used for maintenance, upgrades, or testing:

1. Stop writes to primary
2. Wait for standby to catch up
3. Promote standby
4. Reconfigure applications
5. (Optional) Reconfigure old primary as new standby

### Unplanned Failover

Used when primary fails unexpectedly:

1. Detect primary failure
2. Verify standby is healthy
3. Promote most up-to-date standby
4. Reconfigure applications
5. Investigate old primary

## Failover Verification Script

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

## Failover Tools

Popular tools for automated failover:

| Tool | Type | Features |
|------|------|----------|
| Patroni | Cluster management | Automatic failover, etcd/ZooKeeper |
| repmgr | Replication manager | Manual/auto failover, monitoring |
| pg_auto_failover | Microsoft tool | Simple setup, automatic |
| Stolon | Kubernetes-native | Cloud-native, Consul/etcd |

## Code Examples

```undefined
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