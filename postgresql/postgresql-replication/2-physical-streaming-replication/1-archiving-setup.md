---
source_course: "postgresql-replication"
source_lesson: "postgresql-replication-wal-archiving-setup"
---

# Continuous WAL Archiving

## Introduction

Continuous archiving (also called WAL archiving) creates a persistent copy of WAL files outside the `pg_wal` directory. It is the foundation for both point-in-time recovery and standby servers that need to catch up from a historical position.

This lesson covers how to configure WAL archiving, verify it is working, and handle common failure scenarios.

## Key Concepts

- **archive_mode**: A server parameter that enables WAL archiving. Requires a restart to change.
- **archive_command**: A shell command executed for each completed WAL segment, responsible for copying it to the archive location.
- **archive_timeout**: Forces a WAL segment switch after the specified number of seconds, ensuring recent changes are archived even during low-activity periods.
- **pg_stat_archiver**: A system view providing statistics about the archiving process, including success and failure counts.

## Real World Context

A financial services company processes transactions throughout the day but has low activity overnight. Without `archive_timeout`, the current WAL segment might not be archived for hours during quiet periods, creating a window of potential data loss. Setting `archive_timeout = 300` ensures that even during off-peak hours, WAL is archived at least every five minutes.

## Deep Dive

Enable archiving in `postgresql.conf` with these settings:

```sql
-- Required settings
archive_mode = on
archive_command = 'cp %p /archive/wal/%f'
archive_timeout = 300    -- Force archive every 5 minutes

-- %p = full path to WAL file
-- %f = WAL file name only
```

The `archive_command` receives the full path (`%p`) and filename (`%f`) as placeholders. The command must return exit code 0 only on success.

For production, use more robust archiving strategies:

```bash
# Using rsync for remote archiving
archive_command = 'rsync -a %p archivehost:/archive/wal/%f'

# With compression
archive_command = 'gzip -c %p > /archive/wal/%f.gz'

# Using pgBackRest (recommended for production)
archive_command = 'pgbackrest --stanza=main archive-push %p'
```

Each approach has trade-offs: `cp` is simple but local-only, `rsync` handles remote archiving, and pgBackRest provides compression, encryption, and parallel operations.

Verify that archiving is working correctly with these queries:

```sql
-- Check archive status
SELECT * FROM pg_stat_archiver;

-- Force a WAL switch for testing
SELECT pg_switch_wal();

-- Check last archived WAL
SELECT last_archived_wal, last_archived_time
FROM pg_stat_archiver;
```

If `failed_count` is increasing, check the `last_failed_wal` and `last_failed_time` columns to diagnose the issue.

## Common Pitfalls

- **Archive command that overwrites existing files**: Use `test ! -f /archive/%f && cp %p /archive/%f` to avoid overwriting already-archived segments.
- **Ignoring archive failures**: If `archive_command` fails, WAL files accumulate in `pg_wal` and can fill your disk, potentially causing the entire server to halt.
- **Not testing archive recovery**: Always verify that archived WAL files can actually be restored. An archive that cannot be read is useless.

## Best Practices

- Use a dedicated archive tool like pgBackRest or barman rather than raw `cp` or `rsync` commands in production.
- Set `archive_timeout` to 5-10 minutes to limit potential data loss during low-activity periods.
- Monitor `pg_stat_archiver` and alert on increasing `failed_count`.

## Summary

- WAL archiving copies completed WAL segments to a persistent location outside `pg_wal`.
- Configure it with `archive_mode`, `archive_command`, and `archive_timeout` in `postgresql.conf`.
- The `archive_command` must return exit code 0 only on success and should not overwrite existing files.
- Monitor archiving health via `pg_stat_archiver` and alert on failures.
- Use production-grade tools like pgBackRest for reliable archiving with compression and encryption.

## Code Examples

**Complete Archive Configuration**

```sql
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