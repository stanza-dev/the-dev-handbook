---
source_course: "postgresql-replication"
source_lesson: "pitr-backup-strategy-lesson"
---

# Backup Strategy for PITR

A robust PITR strategy requires careful planning of base backups and WAL archiving.

## WAL Archiving Configuration

```ini
# postgresql.conf - Production settings
wal_level = replica
archive_mode = on
archive_command = 'test ! -f /archive/%f && cp %p /archive/%f'
archive_timeout = 300  # Archive every 5 minutes even if WAL not full
```

## Taking Base Backups

```bash
# Basic backup
pg_basebackup -h primary -D /backup/base -Ft -z -P

# With checkpoint and WAL inclusion
pg_basebackup -h primary \
    -D /backup/base_$(date +%Y%m%d) \
    --checkpoint=fast \
    --wal-method=stream \
    -Ft -z -P

# Backup with manifest for verification
pg_basebackup -h primary -D /backup/base \
    --manifest-checksums=SHA256 \
    --progress
```

## Backup Scheduling

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

## Verifying WAL Archiving

```sql
-- Check archive status
SELECT * FROM pg_stat_archiver;

-- Verify archiving is working
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

## Storage Considerations

```bash
# Estimate WAL growth
# Check current WAL generation rate
SELECT 
    pg_wal_lsn_diff(pg_current_wal_lsn(), '0/0') / 
    (extract(epoch from (now() - pg_postmaster_start_time()))) 
    AS bytes_per_second;

# Typical: 1-10 MB/s for active systems
# Storage needed: rate × retention_period
```

📖 [pg_basebackup](https://www.postgresql.org/docs/18/app-pgbasebackup.html)

## Resources

- [pg_basebackup Documentation](https://www.postgresql.org/docs/18/app-pgbasebackup.html) — Base backup utility reference

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*