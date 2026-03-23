---
source_course: "postgresql-replication"
source_lesson: "postgresql-replication-pitr-backup-strategy"
---

# Backup Strategy for PITR

## Introduction

A robust PITR strategy requires careful planning of base backup schedules, WAL archiving configuration, retention policies, and regular verification. Without all four elements, your recovery capability is incomplete.

This lesson covers production-grade backup configurations, scheduling scripts, verification procedures, and storage planning.

## Key Concepts

- **Base Backup Frequency**: How often you take full cluster backups. More frequent backups mean faster recovery (less WAL to replay).
- **WAL Archive Retention**: How long you keep archived WAL segments. This determines how far back you can recover.
- **archive_timeout**: Forces a WAL segment switch, ensuring recent changes are archived even during low-activity periods.
- **Backup Verification**: The process of confirming that backups and WAL archives are complete and usable for recovery.

## Real World Context

A healthcare application stores patient records that must be recoverable for compliance purposes. The operations team takes daily base backups, archives WAL continuously, and retains 30 days of archives. Every week, they restore the latest backup to a test server and verify data integrity. This process caught a corrupted WAL segment early, allowing them to fix the archiving pipeline before it affected their recovery capability.

## Deep Dive

Configure WAL archiving for production:

```sql
-- postgresql.conf - Production settings
wal_level = replica
archive_mode = on
archive_command = 'test ! -f /archive/%f && cp %p /archive/%f'
archive_timeout = 300  -- Archive every 5 minutes even if WAL not full
```

The `test ! -f` guard prevents overwriting already-archived segments, which is critical for data integrity.

Take base backups with pg_basebackup:

```bash
# Production backup with verification manifest
pg_basebackup -h primary \
    -D /backup/base_$(date +%Y%m%d) \
    --checkpoint=fast \
    --wal-method=stream \
    --manifest-checksums=SHA256 \
    -Ft -z -P
```

The `--manifest-checksums` option creates a verification manifest that can be used later to confirm backup integrity.

Automate backups with a scheduling script:

```bash
#!/bin/bash
# /opt/scripts/daily_backup.sh

BACKUP_DIR="/backup/$(date +%Y%m%d_%H%M%S)"
ARCHIVE_DIR="/archive"

# Take base backup
pg_basebackup -h localhost -U replicator \
    -D "$BACKUP_DIR" \
    --checkpoint=fast \
    --wal-method=stream \
    -Ft -z -P

# Record backup info
echo "Backup completed: $(date)" >> "$BACKUP_DIR/backup.log"
echo "Latest WAL: $(ls -t $ARCHIVE_DIR | head -1)" >> "$BACKUP_DIR/backup.log"

# Cleanup old backups (keep 7 days)
find /backup -maxdepth 1 -type d -mtime +7 -exec rm -rf {} \;
```

This script creates timestamped backups, records metadata, and cleans up old backups automatically.

Verify archiving is working continuously:

```sql
-- Check archive status
SELECT
    archived_count,
    last_archived_wal,
    last_archived_time,
    failed_count,
    last_failed_wal,
    last_failed_time
FROM pg_stat_archiver;

-- Force WAL switch (for testing)
SELECT pg_switch_wal();
```

Alert on any increase in `failed_count` and investigate `last_failed_wal` immediately.

Estimate storage needs based on WAL generation rate:

```sql
-- Estimate WAL generation rate
SELECT
    pg_wal_lsn_diff(pg_current_wal_lsn(), '0/0') /
    (extract(epoch from (now() - pg_postmaster_start_time())))
    AS bytes_per_second;
-- Typical: 1-10 MB/s for active systems
-- Storage needed: rate x retention_period
```

This helps you plan storage capacity for both WAL archives and base backups.

## Common Pitfalls

- **Not testing recovery regularly**: A backup you have never tested might fail when you need it most. Schedule monthly recovery tests.
- **Insufficient retention**: Keeping only 24 hours of WAL means you cannot recover from a bug discovered the next day.
- **No monitoring of archive failures**: A silently failing archive_command means you are accumulating WAL in pg_wal without actually archiving it.

## Best Practices

- Take daily base backups and retain them for at least one week.
- Set `archive_timeout = 300` to ensure WAL is archived at least every five minutes.
- Verify backups weekly by restoring to a test environment and running integrity checks.

## Summary

- A complete PITR strategy combines regular base backups with continuous WAL archiving.
- Use `archive_timeout` to limit the maximum data loss window during low-activity periods.
- Automate backup scheduling, retention cleanup, and verification.
- Monitor `pg_stat_archiver` for failures and alert immediately.
- Estimate storage needs based on WAL generation rate and retention policy.

## Code Examples

**Production Backup Script**

```bash
#!/bin/bash
# Daily backup with verification
pg_basebackup -h localhost -U replicator \
    -D "/backup/$(date +%Y%m%d)" \
    --checkpoint=fast \
    --wal-method=stream \
    --manifest-checksums=SHA256 \
    -Ft -z -P

# Verify backup integrity
pg_verifybackup "/backup/$(date +%Y%m%d)"
```


## Resources

- [pg_basebackup Documentation](https://www.postgresql.org/docs/18/app-pgbasebackup.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*