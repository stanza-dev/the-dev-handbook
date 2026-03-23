---
source_course: "postgresql-replication"
source_lesson: "postgresql-replication-pg-rewind"
---

# Recovering Old Primaries with pg_rewind

## Introduction

After a failover, the old primary has diverged from the new primary's timeline. It cannot simply be restarted as a standby because it contains WAL that was never replicated. `pg_rewind` solves this by synchronizing the old primary's data directory with the new primary, allowing it to rejoin the cluster as a standby without a full `pg_basebackup`.

This lesson covers how pg_rewind works, when to use it, and the step-by-step process for recovering an old primary.

## Key Concepts

- **pg_rewind**: A tool that synchronizes a PostgreSQL data directory with another copy of the same cluster that has diverged, by copying only changed blocks.
- **Timeline Divergence**: After promotion, the new primary starts a new timeline. The old primary's data directory is on the old timeline and contains changes not present on the new primary.
- **wal_log_hints**: A server parameter that must be enabled (or data checksums must be on) for pg_rewind to detect which blocks have changed.
- **Block-Level Synchronization**: pg_rewind compares data files and copies only the blocks that differ, making it much faster than a full pg_basebackup.

## Real World Context

After an automated failover at 3 AM, the old primary comes back online the next morning. It has a 2 TB database, and running pg_basebackup would take hours. With pg_rewind, only the blocks that diverged (typically a small fraction of the total data) are copied from the new primary, and the old primary is ready to rejoin as a standby in minutes rather than hours.

## Deep Dive

### Prerequisites

pg_rewind requires one of these to detect changed blocks:

```sql
-- Option 1: Enable wal_log_hints (requires restart)
wal_log_hints = on

-- Option 2: Enable data checksums at initdb time
-- initdb --data-checksums -D /data

-- Verify checksums
SHOW data_checksums;
```

Without either setting, pg_rewind cannot determine which blocks have changed and will refuse to run.

### How pg_rewind Works

1. Identifies the point where the timelines diverged
2. Scans WAL on the old primary from the divergence point
3. Determines which data blocks were modified after divergence
4. Copies those blocks from the new primary
5. Copies any additional WAL needed for recovery

The result is a data directory that matches the new primary's state at some recent point, ready to start as a standby.

### Step-by-Step Recovery

First, ensure the old primary is stopped:

```bash
# Stop the old primary (if it is running)
pg_ctl stop -D /var/lib/postgresql/data -m fast
```

Run pg_rewind against the new primary:

```bash
# Rewind old primary to match new primary
pg_rewind \
  --target-pgdata=/var/lib/postgresql/data \
  --source-server='host=new-primary port=5432 user=replicator dbname=postgres' \
  --progress
```

pg_rewind will report the number of blocks copied and the total data compared.

Configure the rewound server as a standby:

```bash
# Create standby.signal
touch /var/lib/postgresql/data/standby.signal
```

Update the connection to point to the new primary:

```sql
-- In postgresql.auto.conf
primary_conninfo = 'host=new-primary port=5432 user=replicator'
primary_slot_name = 'old_primary_slot'
```

Finally, start the server:

```bash
# Start as standby
pg_ctl start -D /var/lib/postgresql/data

# Verify it is in recovery mode
psql -c "SELECT pg_is_in_recovery();"  -- Should return true
```

The old primary will replay WAL from the new primary to catch up.

### When pg_rewind Cannot Be Used

pg_rewind fails when:
- WAL between the divergence point and the old primary's shutdown is missing
- wal_log_hints is off and data checksums are not enabled
- The data directory is corrupted

In these cases, a full pg_basebackup is required.

## Common Pitfalls

- **Not enabling wal_log_hints or data checksums beforehand**: This must be configured before the failover happens. Enabling it after does not help for the current divergence.
- **Running pg_rewind while the old primary is still running**: The old primary must be stopped cleanly before pg_rewind can operate on its data directory.
- **Forgetting to create standby.signal after rewind**: Without it, the server starts as a primary instead of a standby, potentially causing split-brain.

## Best Practices

- Always enable `wal_log_hints = on` or data checksums on all servers to ensure pg_rewind is available when needed.
- Test pg_rewind as part of your regular failover drills to verify it works with your data volume.
- After pg_rewind, verify the server is in recovery mode before allowing client connections.

## Summary

- pg_rewind synchronizes a diverged data directory with the new primary by copying only changed blocks.
- It requires `wal_log_hints = on` or data checksums to detect which blocks changed.
- The process is dramatically faster than pg_basebackup for large databases with small divergence.
- After rewinding, configure the server as a standby with `standby.signal` and `primary_conninfo`.
- Always enable wal_log_hints proactively so pg_rewind is available when you need it.

## Code Examples

**pg_rewind Recovery Process**

```bash
# 1. Stop old primary
pg_ctl stop -D /var/lib/postgresql/data -m fast

# 2. Rewind to match new primary
pg_rewind \
  --target-pgdata=/var/lib/postgresql/data \
  --source-server='host=new-primary port=5432 user=replicator dbname=postgres' \
  --progress

# 3. Configure as standby
touch /var/lib/postgresql/data/standby.signal
# Update primary_conninfo in postgresql.auto.conf

# 4. Start as standby
pg_ctl start -D /var/lib/postgresql/data
```


## Resources

- [pg_rewind](https://www.postgresql.org/docs/current/app-pgrewind.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*