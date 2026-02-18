---
source_course: "postgresql-replication"
source_lesson: "logical-replication-failover"
---

# Logical Replication Failover

PostgreSQL 17+ introduced logical replication slot failover, allowing subscriptions to continue working after a physical failover.

## The Problem

Before PostgreSQL 17, when a primary with logical replication failed over:
- Replication slots existed only on old primary
- Subscriptions couldn't connect to new primary
- Manual intervention required to recreate slots

## Failover-Enabled Slots

```sql
-- Create subscription with failover support
CREATE SUBSCRIPTION my_sub
    CONNECTION 'host=primary port=5432 dbname=source'
    PUBLICATION my_pub
    WITH (failover = true);

-- Or create slot directly with failover
SELECT pg_create_logical_replication_slot(
    'my_slot', 
    'pgoutput', 
    false,      -- temporary
    false,      -- two_phase
    true        -- failover enabled
);
```

## Synchronizing Slots to Standby

The standby needs a replication slot for each failover-enabled slot:

```sql
-- On primary: Find slots needing sync
SELECT slot_name, failover, synced
FROM pg_replication_slots
WHERE failover = true;

-- On standby: Create slotsync worker
-- Requires hot_standby_feedback = on
-- And sync_replication_slots = on (postgresql.conf)
```

## Verifying Slot Synchronization

```sql
-- Check sync status on standby
SELECT slot_name, 
       synced, 
       temporary,
       (synced AND NOT temporary AND invalidation_reason IS NULL) AS failover_ready
FROM pg_replication_slots;
```

## Failover Procedure

1. Promote standby to primary
2. Subscribers automatically reconnect to new primary
3. Replication continues from synchronized position

## Code Examples

```undefined
-- On PRIMARY: Create failover-enabled publication
CREATE PUBLICATION failover_pub FOR ALL TABLES;

-- On SUBSCRIBER: Create failover-aware subscription
CREATE SUBSCRIPTION failover_sub
    CONNECTION 'host=primary port=5432 dbname=source user=replicator'
    PUBLICATION failover_pub
    WITH (failover = true, copy_data = false);

-- On STANDBY postgresql.conf:
-- hot_standby_feedback = on
-- sync_replication_slots = on

-- Verify on standby
SELECT slot_name, synced, failover FROM pg_replication_slots;
```


## Resources

- [Logical Replication Failover](https://www.postgresql.org/docs/current/logical-replication-failover.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*