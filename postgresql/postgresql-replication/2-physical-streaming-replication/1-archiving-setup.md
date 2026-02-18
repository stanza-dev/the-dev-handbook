---
source_course: "postgresql-replication"
source_lesson: "wal-archiving-setup"
---

# Continuous WAL Archiving

Continuous archiving (also called WAL archiving) is the foundation for both point-in-time recovery and standby servers. It creates a continuous backup of WAL files that can be used to replay transactions.

## Why Archive WAL?

1. **Disaster Recovery**: Restore to any point in time
2. **Standby Setup**: Standbys can catch up from archives
3. **Long Retention**: Keep history beyond pg_wal
4. **Backup Integrity**: Ensure backups can be restored

## Configuring WAL Archiving

Enable archiving in `postgresql.conf`:

```sql
-- Required settings
archive_mode = on
archive_command = 'cp %p /archive/wal/%f'
archive_timeout = 300    -- Force archive every 5 minutes

-- %p = full path to WAL file
-- %f = WAL file name only
```

## Production Archive Commands

For production, use more robust archiving:

```bash
# Using rsync for remote archiving
archive_command = 'rsync -a %p archivehost:/archive/wal/%f'

# With compression
archive_command = 'gzip -c %p > /archive/wal/%f.gz'

# Using pgBackRest (recommended)
archive_command = 'pgbackrest --stanza=main archive-push %p'
```

## Verifying Archiving

```sql
-- Check archive status
SELECT * FROM pg_stat_archiver;

-- Force a WAL switch for testing
SELECT pg_switch_wal();

-- Check last archived WAL
SELECT last_archived_wal, last_archived_time 
FROM pg_stat_archiver;
```

**Warning:** If `archive_command` fails, WAL files accumulate in `pg_wal` and can fill your disk!

## Code Examples

```undefined
-- postgresql.conf for production archiving
archive_mode = on
archive_command = 'test ! -f /archive/wal/%f && cp %p /archive/wal/%f'
archive_timeout = 300
wal_level = replica
```


## Resources

- [Continuous Archiving](https://www.postgresql.org/docs/current/continuous-archiving.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*