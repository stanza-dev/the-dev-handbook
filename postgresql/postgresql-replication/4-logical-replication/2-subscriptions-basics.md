---
source_course: "postgresql-replication"
source_lesson: "subscriptions-basics"
---

# Creating Subscriptions

Subscriptions are created on the subscriber (target) server and connect to publications on the publisher. They define how data is pulled and applied.

## Creating Subscriptions

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

## Initial Data Synchronization

By default, `copy_data = true` performs an initial table sync:

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

## Managing Subscriptions

```sql
-- Disable subscription (stop replication)
ALTER SUBSCRIPTION sub_orders DISABLE;

-- Enable subscription
ALTER SUBSCRIPTION sub_orders ENABLE;

-- Refresh publication (detect new tables)
ALTER SUBSCRIPTION sub_orders REFRESH PUBLICATION;

-- Change connection
ALTER SUBSCRIPTION sub_orders 
    CONNECTION 'host=new-publisher port=5432 dbname=shop';

-- Drop subscription
DROP SUBSCRIPTION sub_orders;
```

## Monitoring Subscriptions

```sql
-- View subscription status
SELECT subname, subenabled, subconninfo
FROM pg_subscription;

-- View replication lag
SELECT * FROM pg_stat_subscription;
```

## Code Examples

```undefined
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

-- Monitor subscription progress
SELECT subname, received_lsn, latest_end_lsn, last_msg_send_time
FROM pg_stat_subscription;
```


## Resources

- [CREATE SUBSCRIPTION](https://www.postgresql.org/docs/current/sql-createsubscription.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*