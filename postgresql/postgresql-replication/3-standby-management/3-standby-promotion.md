---
source_course: "postgresql-replication"
source_lesson: "postgresql-replication-standby-promotion"
---

# Promoting a Standby to Primary

## Introduction

When the primary server fails, a standby must be promoted to become the new primary. PostgreSQL provides multiple promotion methods, each suited to different operational contexts.

This lesson covers promotion mechanics, timeline behavior, verification steps, and how to reconnect remaining standbys to the new primary.

## Key Concepts

- **Promotion**: The process of converting a standby server into a read-write primary server.
- **Timeline**: A numeric identifier that changes during promotion, preventing the old and new primaries from being confused.
- **pg_promote()**: A SQL function available on the standby that triggers promotion programmatically.
- **pg_ctl promote**: A command-line tool for promoting a standby when SQL access is not available.

## Real World Context

During a hardware failure at 3 AM, the on-call engineer receives an alert that the primary database is unreachable. They verify the standby is healthy and has minimal lag, then execute `SELECT pg_promote(wait := true, wait_seconds := 60)`. Within seconds, the standby becomes the new primary and the application recovers. The next morning, the team reconfigures the repaired old primary as a standby of the new primary.

## Deep Dive

PostgreSQL supports two promotion methods:

```bash
# Method 1: Using pg_ctl from the command line
pg_ctl promote -D /var/lib/postgresql/data
```

This is useful when you cannot connect to the standby via SQL.

```sql
-- Method 2: Using SQL (requires connection to standby)
SELECT pg_promote();

-- With timeout (wait up to 60 seconds for completion)
SELECT pg_promote(wait := true, wait_seconds := 60);
```

The SQL method is preferred when you have a connection, as it can wait for promotion to complete before returning.

During promotion, the following sequence occurs:

1. Standby stops accepting WAL from the primary
2. Remaining WAL in the receive buffer is replayed
3. `standby.signal` is removed
4. A new timeline is created (prevents confusion with old primary)
5. Server becomes read-write

Verify the promotion was successful:

```sql
-- Before promotion (standby)
SELECT pg_is_in_recovery();  -- Returns true

-- After promotion (new primary)
SELECT pg_is_in_recovery();  -- Returns false

-- Check timeline
SELECT timeline_id FROM pg_control_checkpoint();
```

The timeline ID increments with each promotion, providing a clear audit trail.

After promotion, reconnect other standbys to the new primary:

```sql
-- Update primary_conninfo to point to new primary
ALTER SYSTEM SET primary_conninfo = 'host=new-primary port=5432 user=replicator';
SELECT pg_reload_conf();
```

Alternatively, restart the standby after editing `postgresql.auto.conf`.

The old primary must NOT be restarted without first being reconfigured as a standby, or data divergence will occur.

## Common Pitfalls

- **Promoting a standby with significant lag**: Always check replication lag before promotion. Promoting a standby that is minutes behind means losing all transactions in that gap.
- **Restarting the old primary as a primary**: If the old primary comes back online without being reconfigured, you will have two independent primaries (split-brain), causing data divergence.
- **Forgetting to update other standbys**: After promotion, remaining standbys still point to the old primary and will not receive new WAL until reconfigured.

## Best Practices

- Always check replication lag before promoting: ensure `pg_wal_lsn_diff(pg_last_wal_receive_lsn(), pg_last_wal_replay_lsn())` is near zero.
- Use `pg_promote(wait := true)` to ensure promotion completes before proceeding with application failover.
- Document the promotion procedure and test it regularly with planned switchovers.

## Summary

- Promote a standby using `pg_promote()` (SQL) or `pg_ctl promote` (command line).
- Promotion creates a new timeline, stops WAL replay, removes `standby.signal`, and enables read-write.
- Verify promotion with `pg_is_in_recovery()` returning false.
- Reconnect remaining standbys to the new primary and never restart the old primary without reconfiguring it.
- Always check lag before promotion to minimize data loss.

## Code Examples

**Promotion and Verification**

```sql
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