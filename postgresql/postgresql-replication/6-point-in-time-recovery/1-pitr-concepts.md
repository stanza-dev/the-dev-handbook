---
source_course: "postgresql-replication"
source_lesson: "postgresql-replication-pitr-concepts"
---

# Understanding PITR Concepts

## Introduction

Point-in-Time Recovery (PITR) allows you to restore a database to any moment in time, providing protection against data loss from accidental deletions, corruption, or application bugs.

This lesson introduces the core PITR concepts: how it works, what components it requires, and the different recovery target types available.

## Key Concepts

- **Base Backup**: A full physical copy of the database cluster, serving as the starting point for recovery.
- **WAL Archive**: A continuous stream of all changes since the base backup, stored outside pg_wal for long-term retention.
- **Recovery Target**: The exact point in time, transaction, or named restore point to which you want to recover.
- **Recovery Timeline**: A numeric identifier that changes during recovery, creating a branching history of the database.

## Real World Context

A developer accidentally runs `DELETE FROM orders WHERE status = 'pending'` without a WHERE clause refinement, deleting thousands of valid orders. With PITR, the DBA restores the database to one minute before the erroneous DELETE was executed, recovering all the lost data. Without PITR, the only option would be the last nightly backup, potentially losing a full day of transactions.

## Deep Dive

PITR relies on three components working together:

1. **Base Backup**: A full copy of the database cluster
2. **WAL Archives**: A continuous stream of all changes since the backup
3. **Recovery Target**: The exact point you want to restore to

```
[Base Backup] -> [WAL1] -> [WAL2] -> ... -> [WALn] -> [Recovery Target]
     ^                                                      ^
  Starting point                                       Destination
```

The recovery process replays WAL from the base backup forward until reaching the specified target.

PITR supports multiple recovery target types, each useful in different situations:

| Target Type | Use Case | Precision |
|-------------|----------|-----------|
| `recovery_target_time` | Restore to specific timestamp | Microsecond |
| `recovery_target_xid` | Restore to transaction ID | Transaction |
| `recovery_target_name` | Restore to named point | Custom |
| `recovery_target_lsn` | Restore to WAL position | Byte-level |

The prerequisites for PITR must be configured before you need it:

```sql
-- postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'cp %p /archive/%f'
```

These settings ensure WAL is archived continuously, providing the change history needed for recovery.

Named restore points let you mark important moments:

```sql
-- Create a named restore point before risky operations
SELECT pg_create_restore_point('before_schema_migration');

-- Perform migration...
ALTER TABLE orders ADD COLUMN new_field TEXT;

-- If something goes wrong, you can restore to this exact point
```

Restore points are embedded in the WAL stream and can be used as recovery targets.

Comparing PITR to traditional backups:

| Feature | Traditional Backup | PITR |
|---------|-------------------|------|
| Recovery Point | Backup time only | Any point in time |
| Data Loss Window | Hours/days | Seconds |
| Storage | Full backups | Base + incremental WAL |
| Complexity | Simple | Moderate |

## Common Pitfalls

- **Not configuring archiving before you need it**: PITR requires continuous WAL archiving. If archiving was not enabled when the problem occurred, you cannot recover.
- **Not creating restore points before risky operations**: Named restore points provide precise recovery targets. Without them, you must estimate the timestamp of the problem.
- **Assuming PITR can recover from hardware corruption**: PITR replays WAL, so if the WAL itself or the base backup is corrupted, recovery will fail.

## Best Practices

- Enable WAL archiving on every production database, even if you do not think you need PITR today.
- Create named restore points before migrations, bulk operations, and deployments.
- Regularly test PITR by restoring to a test server to verify your archives are complete and usable.

## Summary

- PITR restores a database to any point in time using a base backup plus archived WAL.
- Four recovery target types offer precision from timestamp-level to byte-level.
- WAL archiving must be configured proactively, before you need recovery.
- Named restore points provide precise recovery markers for planned operations.
- Regular PITR testing is essential to verify backup integrity.

## Resources

- [Continuous Archiving and Point-in-Time Recovery](https://www.postgresql.org/docs/18/continuous-archiving.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*