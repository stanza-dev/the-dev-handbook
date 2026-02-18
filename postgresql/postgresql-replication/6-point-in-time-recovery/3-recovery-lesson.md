---
source_course: "postgresql-replication"
source_lesson: "pitr-recovery-lesson"
---

# Performing PITR Recovery

Step-by-step guide to recovering a database to a specific point in time.

## Recovery Steps Overview

1. Stop PostgreSQL (if running)
2. Restore base backup
3. Configure recovery settings
4. Create recovery.signal
5. Start PostgreSQL
6. Verify recovery

## Step 1: Prepare Recovery Environment

```bash
# Stop PostgreSQL
sudo systemctl stop postgresql

# Backup current data directory (safety)
mv /var/lib/postgresql/18/main /var/lib/postgresql/18/main.failed

# Create new data directory
mkdir /var/lib/postgresql/18/main
```

## Step 2: Restore Base Backup

```bash
# If tar format
tar -xzf /backup/base.tar.gz -C /var/lib/postgresql/18/main/

# If plain format
cp -R /backup/base/* /var/lib/postgresql/18/main/

# Set permissions
chown -R postgres:postgres /var/lib/postgresql/18/main
chmod 700 /var/lib/postgresql/18/main
```

## Step 3: Configure Recovery

```ini
# postgresql.conf (in data directory)
restore_command = 'cp /archive/%f %p'
recovery_target_time = '2024-01-15 14:30:00'
recovery_target_action = 'promote'

# Optional: more specific targeting
# recovery_target_name = 'before_schema_migration'
# recovery_target_xid = '12345'
# recovery_target_timeline = 'latest'
```

## Step 4: Create Recovery Signal

```bash
# This file triggers recovery mode
touch /var/lib/postgresql/18/main/recovery.signal
```

## Step 5: Start Recovery

```bash
sudo systemctl start postgresql

# Monitor recovery progress
tail -f /var/log/postgresql/postgresql-18-main.log
```

## Step 6: Verify Recovery

```sql
-- Check timeline
SELECT pg_is_in_recovery();
-- Should return false after recovery completes

-- Verify data state
SELECT * FROM orders WHERE created_at > '2024-01-15 14:30:00';
-- Should return no rows if recovered correctly

-- Check WAL position
SELECT pg_current_wal_lsn();
```

## Recovery Target Actions

| Action | Behavior |
|--------|----------|
| `pause` | Pause at recovery target, allowing inspection |
| `promote` | Promote to primary and accept writes |
| `shutdown` | Shut down after reaching target |

```ini
# Pause for inspection
recovery_target_action = 'pause'

# After verifying, resume manually:
# SELECT pg_wal_replay_resume();
```

📖 [Recovery Configuration](https://www.postgresql.org/docs/18/runtime-config-wal.html#RUNTIME-CONFIG-WAL-RECOVERY-TARGET)

## Resources

- [Recovery Configuration](https://www.postgresql.org/docs/18/runtime-config-wal.html#RUNTIME-CONFIG-WAL-RECOVERY-TARGET) — Recovery target settings

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*