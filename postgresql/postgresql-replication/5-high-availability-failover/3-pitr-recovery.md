---
source_course: "postgresql-replication"
source_lesson: "pitr-recovery"
---

# Point-in-Time Recovery

Point-in-Time Recovery (PITR) allows restoring a database to any moment in the past, using a base backup plus WAL archives.

## PITR Requirements

1. A base backup (from pg_basebackup)
2. All WAL archives from backup time to target time
3. Knowledge of target recovery point

## Recovery Configuration

Create `recovery.signal` and configure `postgresql.conf`:

```sql
-- postgresql.conf for PITR
restore_command = 'cp /archive/wal/%f %p'
recovery_target_time = '2024-03-15 14:30:00'
recovery_target_action = 'promote'  -- or 'pause' or 'shutdown'
```

## Recovery Target Options

```sql
-- Recover to specific time
recovery_target_time = '2024-03-15 14:30:00 UTC'

-- Recover to specific transaction
recovery_target_xid = '12345'

-- Recover to named restore point
recovery_target_name = 'before_migration'

-- Recover to specific LSN
recovery_target_lsn = '0/1234567'

-- Include or exclude the target
recovery_target_inclusive = true  -- default
```

## Creating Restore Points

```sql
-- Create named restore point before risky operation
SELECT pg_create_restore_point('before_schema_change');

-- Now perform migration...
ALTER TABLE users ADD COLUMN preferences jsonb;

-- If something goes wrong, restore to 'before_schema_change'
```

## PITR Procedure

```bash
# 1. Stop PostgreSQL if running
pg_ctl stop -D /data

# 2. Move or remove current data directory
mv /data /data.old

# 3. Restore base backup
pg_basebackup -D /data  # Or extract from tar backup

# 4. Configure recovery
touch /data/recovery.signal
# Edit postgresql.conf with restore_command and target

# 5. Start PostgreSQL - recovery begins automatically
pg_ctl start -D /data
```

## Code Examples

```undefined
-- postgresql.conf for point-in-time recovery
restore_command = 'cp /mnt/wal_archive/%f %p'
recovery_target_time = '2024-03-15 14:30:00 UTC'
recovery_target_timeline = 'latest'
recovery_target_action = 'promote'

-- Alternative: recover to restore point
-- recovery_target_name = 'before_data_migration'
-- recovery_target_inclusive = false  -- Stop just before the point
```


## Resources

- [Recovery Target Settings](https://www.postgresql.org/docs/current/recovery-target-settings.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*