---
source_course: "postgresql-replication"
source_lesson: "postgresql-replication-subscriptions"
---

# Creating Subscriptions

## Introduction

Subscriptions are created on the subscriber (target) server and connect to publications on the publisher. They define how data is pulled, applied, and monitored.

This lesson covers subscription creation, initial synchronization, management, and the new PostgreSQL 18 defaults and conflict monitoring capabilities.

## Key Concepts

- **Subscription**: A subscriber-side object that connects to a publication on the publisher, pulling and applying replicated changes.
- **copy_data**: Controls whether the subscription performs an initial bulk copy of existing data when created.
- **streaming (PG18 default: parallel)**: Controls whether large transactions are streamed before they complete. PostgreSQL 18 changes the default from `off` to `parallel`.
- **pg_stat_subscription_stats**: A statistics view that, in PostgreSQL 18, gains new columns for monitoring replication conflicts.

## Real World Context

A multi-region application creates subscriptions in each regional database to receive product catalog updates from the central publisher. With PostgreSQL 18's default `streaming = parallel`, large catalog updates that add thousands of products are applied incrementally as they happen, rather than waiting for the entire transaction to commit on the publisher. This reduces replication lag for large batch operations.

## Deep Dive

Create a basic subscription:

```sql
-- Basic subscription
CREATE SUBSCRIPTION sub_orders
    CONNECTION 'host=publisher port=5432 dbname=shop user=replicator password=secret'
    PUBLICATION pub_orders;

-- Subscription without initial data copy
CREATE SUBSCRIPTION sub_orders
    CONNECTION 'host=publisher port=5432 dbname=shop user=replicator'
    PUBLICATION pub_orders
    WITH (copy_data = false);

-- Multiple publications
CREATE SUBSCRIPTION sub_all
    CONNECTION 'host=pub1 port=5432 dbname=shop'
    PUBLICATION pub_orders, pub_inventory;
```

The `copy_data = false` option is useful when the subscriber already has the data or when you want to synchronize data through other means.

### Streaming Default Change (PostgreSQL 18)

PostgreSQL 18 changes the default `streaming` option from `off` to `parallel`:

```sql
-- PG18: streaming = parallel is now the default
CREATE SUBSCRIPTION sub_orders
    CONNECTION 'host=publisher port=5432 dbname=shop user=replicator'
    PUBLICATION pub_orders;
    -- streaming = parallel implied

-- Explicitly set to off if needed for backward compatibility
CREATE SUBSCRIPTION sub_orders
    CONNECTION 'host=publisher port=5432 dbname=shop user=replicator'
    PUBLICATION pub_orders
    WITH (streaming = off);
```

With `streaming = parallel`, large transactions are applied on the subscriber before they commit on the publisher, using parallel apply workers. This significantly reduces replication lag for bulk operations.

Check initial synchronization status:

```sql
-- Check sync status
SELECT srsubid, srrelid::regclass, srsubstate
FROM pg_subscription_rel;

-- srsubstate values:
-- 'i' = initializing
-- 'd' = data is being copied
-- 's' = synchronized
-- 'r' = ready (normal replication)
```

Manage subscriptions after creation:

```sql
-- Disable subscription (stop replication)
ALTER SUBSCRIPTION sub_orders DISABLE;

-- Enable subscription
ALTER SUBSCRIPTION sub_orders ENABLE;

-- Refresh publication (detect new tables)
ALTER SUBSCRIPTION sub_orders REFRESH PUBLICATION;

-- Drop subscription
DROP SUBSCRIPTION sub_orders;
```

### Conflict Monitoring (PostgreSQL 18)

PostgreSQL 18 adds seven new columns to `pg_stat_subscription_stats` for monitoring replication conflicts. Conflicts are also logged to the server log:

```sql
-- Monitor replication conflicts (PG18+)
SELECT subname,
       confl_insert_exists,
       confl_update_origin_differs,
       confl_update_exists,
       confl_update_missing,
       confl_delete_origin_differs,
       confl_delete_missing,
       confl_multiple_unique_conflicts
FROM pg_stat_subscription_stats;
```

These columns help diagnose common issues:
- `confl_insert_exists`: A replicated INSERT violated a unique constraint (row already exists on subscriber)
- `confl_update_missing`: A replicated UPDATE could not find the target row on the subscriber
- `confl_delete_missing`: A replicated DELETE could not find the target row
- `confl_multiple_unique_conflicts`: An operation conflicted with multiple unique constraints

## Common Pitfalls

- **Not refreshing after publication changes**: When tables are added to a publication, existing subscriptions do not see them until you run `ALTER SUBSCRIPTION ... REFRESH PUBLICATION`.
- **Dropping a subscription without disabling first**: Dropping an active subscription can leave orphaned replication slots on the publisher. Disable first, then drop.
- **Ignoring conflict statistics**: In PostgreSQL 18, conflicts are now visible in pg_stat_subscription_stats. Ignoring them can lead to silent data divergence between publisher and subscriber.

## Best Practices

- On PostgreSQL 18+, leverage the default `streaming = parallel` for better large-transaction performance.
- Monitor `pg_stat_subscription_stats` conflict columns regularly, especially after schema changes.
- Use `REFRESH PUBLICATION` after adding tables to keep subscriptions in sync with publication definitions.

## Summary

- Subscriptions connect to publications and apply replicated changes on the subscriber.
- The `copy_data` option controls initial data synchronization; `streaming` controls large-transaction handling.
- PostgreSQL 18 changes the default streaming mode to `parallel`, applying large transactions before they commit on the publisher.
- Seven new conflict monitoring columns in `pg_stat_subscription_stats` provide visibility into replication conflicts.
- Always refresh subscriptions after publication changes and monitor conflict statistics.

## Code Examples

**Subscription with Options and Conflict Monitoring**

```sql
-- Create subscription with custom slot name
CREATE SUBSCRIPTION my_sub
    CONNECTION 'host=publisher dbname=source user=replicator'
    PUBLICATION source_pub
    WITH (
        copy_data = true,
        create_slot = true,
        slot_name = 'my_custom_slot',
        synchronous_commit = off
    );

-- PG18: Monitor conflict statistics
SELECT subname, confl_insert_exists, confl_update_missing, confl_delete_missing
FROM pg_stat_subscription_stats;
```


## Resources

- [CREATE SUBSCRIPTION](https://www.postgresql.org/docs/current/sql-createsubscription.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*