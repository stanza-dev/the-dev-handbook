---
source_course: "postgresql-replication"
source_lesson: "standby-promotion"
---

# Promoting a Standby to Primary

When the primary fails, you need to promote a standby to become the new primary. PostgreSQL provides multiple methods for promotion.

## Promotion Methods

### Using pg_ctl

```bash
# Promote standby to primary
pg_ctl promote -D /var/lib/postgresql/data
```

### Using SQL

```sql
-- From a connection to the standby
SELECT pg_promote();

-- With timeout (wait up to 60 seconds)
SELECT pg_promote(wait := true, wait_seconds := 60);
```

## What Happens During Promotion

1. Standby stops accepting WAL from primary
2. Remaining WAL in receive buffer is replayed
3. `standby.signal` is removed
4. New timeline is created (prevents confusion with old primary)
5. Server becomes read-write

## Verifying Promotion

```sql
-- Before promotion (standby)
SELECT pg_is_in_recovery();  -- Returns true

-- After promotion (new primary)
SELECT pg_is_in_recovery();  -- Returns false

-- Check timeline
SELECT timeline_id FROM pg_control_checkpoint();
```

## Reconnecting Other Standbys

After promotion, other standbys must reconnect to the new primary:

```sql
-- Update primary_conninfo to point to new primary
ALTER SYSTEM SET primary_conninfo = 'host=new-primary port=5432 user=replicator';
SELECT pg_reload_conf();

-- Or restart the standby after editing postgresql.auto.conf
```

**Important:** The old primary should NOT be restarted without reconfiguring as a standby, or data divergence will occur!

## Code Examples

```undefined
-- Step 1: Verify current state
SELECT pg_is_in_recovery();

-- Step 2: Promote
SELECT pg_promote(wait := true, wait_seconds := 120);

-- Step 3: Verify promotion succeeded
SELECT pg_is_in_recovery();  -- Should be false

-- Step 4: Check new timeline
SELECT timeline_id FROM pg_control_checkpoint();
```


## Resources

- [Failover](https://www.postgresql.org/docs/current/warm-standby-failover.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*