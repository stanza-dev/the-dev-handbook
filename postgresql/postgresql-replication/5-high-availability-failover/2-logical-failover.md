---
source_course: "postgresql-replication"
source_lesson: "postgresql-replication-logical-failover"
---

# Logical Replication Failover

## Introduction

PostgreSQL 17+ introduced logical replication slot failover, allowing subscriptions to continue working seamlessly after a physical failover. Previously, logical replication slots existed only on the primary, and failover required manual recreation.

This lesson covers failover-enabled slots, slot synchronization to standbys, and the verification process.

## Key Concepts

- **Failover-Enabled Slot**: A logical replication slot marked with `failover = true`, indicating it should be synchronized to physical standbys.
- **Slot Synchronization**: The process by which a physical standby maintains copies of failover-enabled logical slots from the primary.
- **sync_replication_slots**: A standby parameter that enables the slotsync worker to keep logical slots in sync with the primary.
- **Invalidation**: When a slot cannot be synchronized (e.g., due to WAL removal), it is marked as invalidated and the reason is recorded.

## Real World Context

A company runs a primary PostgreSQL server that publishes order data to several downstream analytics subscribers via logical replication. They also maintain a physical standby for high availability. Before PostgreSQL 17, if the primary failed and the standby was promoted, all logical subscribers lost their replication position and required manual intervention. With failover-enabled slots, the promoted standby already has the correct slot positions, and subscribers reconnect automatically.

## Deep Dive

### The Problem Before PG17

Before PostgreSQL 17, when a primary with logical replication failed over:
- Replication slots existed only on the old primary
- Subscriptions could not connect to the new primary
- Manual intervention was required to recreate slots

### Creating Failover-Enabled Slots

```sql
-- Create subscription with failover support
CREATE SUBSCRIPTION my_sub
    CONNECTION 'host=primary port=5432 dbname=source'
    PUBLICATION my_pub
    WITH (failover = true);

-- Or create a slot directly with failover
SELECT pg_create_logical_replication_slot(
    'my_slot',
    'pgoutput',
    false,      -- temporary
    false,      -- two_phase
    true        -- failover enabled
);
```

The `failover = true` option marks the slot for synchronization to physical standbys.

### Configuring the Standby for Slot Sync

The standby must be configured to synchronize failover-enabled slots:

```sql
-- On standby postgresql.conf
hot_standby_feedback = on
sync_replication_slots = on
```

With these settings, a slotsync worker on the standby periodically copies slot positions from the primary.

### Verifying Slot Synchronization

```sql
-- On primary: Find failover-enabled slots
SELECT slot_name, failover
FROM pg_replication_slots
WHERE failover = true;

-- On standby: Check sync status
SELECT slot_name,
       synced,
       temporary,
       invalidation_reason,
       (synced AND NOT temporary AND invalidation_reason IS NULL) AS failover_ready
FROM pg_replication_slots;
```

A slot is failover-ready when `synced = true`, `temporary = false`, and `invalidation_reason IS NULL`.

### Failover Procedure

1. Promote standby to primary
2. Subscribers automatically reconnect to new primary
3. Replication continues from the synchronized position

No manual slot recreation is needed.

## Common Pitfalls

- **Forgetting sync_replication_slots on the standby**: Without this setting, the slotsync worker does not run and slots are not copied to the standby.
- **Not checking invalidation_reason**: If a slot is invalidated (e.g., due to WAL removal), it will not work after failover. Monitor for invalidated slots.
- **Assuming all slots sync automatically**: Only slots with `failover = true` are synchronized. Existing slots without the flag are not included.

## Best Practices

- Always create logical subscriptions with `failover = true` when the publisher has physical standbys for HA.
- Enable `sync_replication_slots` and `hot_standby_feedback` on all physical standbys.
- Regularly verify that failover-ready slots exist on the standby using the query above.

## Summary

- PostgreSQL 17+ enables logical replication slot synchronization to physical standbys.
- Subscriptions created with `failover = true` have their slots automatically copied to standbys.
- The standby requires `sync_replication_slots = on` and `hot_standby_feedback = on`.
- After promotion, subscribers reconnect automatically without manual slot recreation.
- Monitor slot sync status and invalidation reasons to ensure failover readiness.

## Code Examples

**Logical Slot Failover Setup**

```sql
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