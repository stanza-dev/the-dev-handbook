---
source_course: "postgresql-replication"
source_lesson: "pitr-concepts-lesson"
---

# Understanding Point-in-Time Recovery

Point-in-Time Recovery (PITR) allows you to restore a database to any moment in time, providing protection against data loss from accidental deletions, corruption, or application bugs.

## How PITR Works

PITR relies on three components working together:

1. **Base Backup**: A full copy of the database cluster
2. **WAL Archives**: Continuous stream of all changes since the backup
3. **Recovery Target**: The exact point you want to restore to

```
[Base Backup] → [WAL1] → [WAL2] → ... → [WALn] → [Recovery Target]
     ↑                                              ↑
  Starting point                               Destination
```

## Recovery Granularity

PITR supports multiple recovery targets:

| Target Type | Use Case | Precision |
|-------------|----------|----------|
| `recovery_target_time` | Restore to specific timestamp | Microsecond |
| `recovery_target_xid` | Restore to transaction ID | Transaction |
| `recovery_target_name` | Restore to named point | Custom |
| `recovery_target_lsn` | Restore to WAL position | Byte-level |

## Prerequisites for PITR

```ini
# postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'cp %p /archive/%f'
```

## Creating Named Restore Points

```sql
-- Create a named restore point before risky operations
SELECT pg_create_restore_point('before_schema_migration');

-- Perform migration...
ALTER TABLE orders ADD COLUMN new_field TEXT;

-- If something goes wrong, you can restore to this point
```

## PITR vs Traditional Backup

| Feature | Traditional Backup | PITR |
|---------|-------------------|------|
| Recovery Point | Backup time only | Any point in time |
| Data Loss Window | Hours/days | Seconds |
| Storage | Full backups | Base + incremental WAL |
| Complexity | Simple | Moderate |

📖 [Continuous Archiving and PITR](https://www.postgresql.org/docs/18/continuous-archiving.html)

## Resources

- [Continuous Archiving and Point-in-Time Recovery](https://www.postgresql.org/docs/18/continuous-archiving.html) — Complete PITR guide

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*