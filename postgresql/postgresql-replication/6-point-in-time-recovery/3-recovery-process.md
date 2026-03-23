---
source_course: "postgresql-replication"
source_lesson: "postgresql-replication-pitr-recovery-process"
---

# Performing PITR Recovery

## Introduction

Performing a point-in-time recovery involves restoring a base backup, configuring recovery parameters, creating the recovery signal file, and verifying the result. Each step must be executed carefully to ensure a successful recovery.

This lesson provides a detailed, step-by-step guide for the entire PITR process.

## Key Concepts

- **restore_command**: A parameter specifying the shell command to retrieve archived WAL files during recovery.
- **recovery_target_time**: The timestamp to which you want to recover, specified in PostgreSQL timestamp format.
- **recovery_target_action**: Controls what happens when the recovery target is reached: `promote`, `pause`, or `shutdown`.
- **recovery.signal**: A file in the data directory that triggers recovery mode when PostgreSQL starts.

## Real World Context

At 2 PM, a deployment script accidentally truncates the `customers` table. The DBA is notified at 2:15 PM. They restore last night's base backup, configure recovery to target 1:59 PM (one minute before the truncation), create the recovery.signal file, and start PostgreSQL. Within 30 minutes, the database is restored with all customer data intact. They then use `pg_dump` to extract the recovered table and import it back into the production database.

## Deep Dive

### Recovery Steps Overview

1. Stop PostgreSQL (if running)
2. Restore base backup
3. Configure recovery settings
4. Create recovery.signal
5. Start PostgreSQL
6. Verify recovery

### Step 1: Prepare Recovery Environment

```bash
# Stop PostgreSQL
sudo systemctl stop postgresql

# Backup current data directory (safety net)
mv /var/lib/postgresql/18/main /var/lib/postgresql/18/main.failed

# Create new data directory
mkdir /var/lib/postgresql/18/main
```

Always preserve the failed data directory until you confirm recovery is successful.

### Step 2: Restore Base Backup

```bash
# If tar format
tar -xzf /backup/base.tar.gz -C /var/lib/postgresql/18/main/

# If plain format
cp -R /backup/base/* /var/lib/postgresql/18/main/

# Set correct permissions
chown -R postgres:postgres /var/lib/postgresql/18/main
chmod 700 /var/lib/postgresql/18/main
```

Permissions must be exactly `700` on the data directory, or PostgreSQL will refuse to start.

### Step 3: Configure Recovery

```sql
-- postgresql.conf (in data directory)
restore_command = 'cp /archive/%f %p'
recovery_target_time = '2024-01-15 14:30:00'
recovery_target_action = 'promote'
```

The `restore_command` is the inverse of `archive_command`: it retrieves archived WAL files for replay.

Alternative recovery targets offer different precision:

```sql
-- Recover to named restore point
recovery_target_name = 'before_schema_migration'

-- Recover to specific transaction
recovery_target_xid = '12345'

-- Recover to specific LSN
recovery_target_lsn = '0/1234567'

-- Include or exclude the target
recovery_target_inclusive = true  -- default
recovery_target_timeline = 'latest'
```

Use `recovery_target_inclusive = false` when you want to stop just before the target, not at it.

### Step 4: Create Recovery Signal

```bash
# This file triggers recovery mode
touch /var/lib/postgresql/18/main/recovery.signal
```

PostgreSQL checks for this file at startup and enters recovery mode if it exists.

### Step 5: Start Recovery

```bash
sudo systemctl start postgresql

# Monitor recovery progress
tail -f /var/log/postgresql/postgresql-18-main.log
```

You will see log messages as PostgreSQL replays each WAL segment.

### Step 6: Verify Recovery

```sql
-- Check recovery completed
SELECT pg_is_in_recovery();  -- Should return false after promote

-- Verify data state
SELECT count(*) FROM customers;  -- Confirm data is present

-- Check WAL position
SELECT pg_current_wal_lsn();
```

If `recovery_target_action = 'pause'`, you can inspect the database before committing:

```sql
-- Resume recovery after inspection
SELECT pg_wal_replay_resume();
```

The pause action is useful when you are unsure of the exact recovery target and want to verify data before finalizing.

## Common Pitfalls

- **Wrong permissions on data directory**: PostgreSQL requires exactly `700` permissions on the data directory. Any other permission causes startup failure.
- **Missing WAL segments**: If any WAL segment between the base backup and the recovery target is missing, recovery stops at that gap.
- **Forgetting to remove recovery.signal after testing**: If you restart the server later, it will attempt recovery again if the file is still present. PostgreSQL removes it automatically after successful recovery with `promote` action.

## Best Practices

- Use `recovery_target_action = 'pause'` for critical recoveries so you can verify data before finalizing.
- Always preserve the original (failed) data directory until recovery is confirmed.
- Practice the recovery procedure regularly so it becomes familiar under pressure.

## Summary

- PITR recovery follows six steps: stop, restore backup, configure, create signal, start, verify.
- The `restore_command` retrieves archived WAL; recovery targets specify the destination point.
- Use `pause` action to inspect data before committing to the recovery.
- Correct file permissions and complete WAL archives are essential for successful recovery.
- Regular practice of the recovery procedure builds confidence and catches issues early.

## Code Examples

**PITR Configuration Example**

```sql
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

- [Recovery Configuration](https://www.postgresql.org/docs/18/runtime-config-wal.html#RUNTIME-CONFIG-WAL-RECOVERY-TARGET) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*