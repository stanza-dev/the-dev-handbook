---
source_course: "postgresql-replication"
source_lesson: "pg-basebackup"
---

# Creating Base Backups with pg_basebackup

`pg_basebackup` is the standard tool for creating physical backups and initializing standby servers. It creates a consistent copy of the entire database cluster.

## Basic Usage

```bash
# Simple backup to directory
pg_basebackup -h primary-host -D /backup/base -P -X stream

# Options explained:
# -h: Primary server hostname
# -D: Destination directory
# -P: Show progress
# -X stream: Stream WAL during backup (recommended)
```

## Backup Formats

```bash
# Plain format (default) - ready to start
pg_basebackup -D /backup/base -Fp

# Tar format - compressed, portable
pg_basebackup -D /backup -Ft -z

# With compression
pg_basebackup -D /backup -Ft --compress=zstd:level=5
```

## Setting Up for Standby

To create a standby-ready backup:

```bash
# Create backup with standby signal
pg_basebackup -h primary -D /var/lib/postgresql/data \
  -X stream \
  -P \
  -R    # Creates standby.signal and configures primary_conninfo

# Using a replication slot (prevents WAL removal)
pg_basebackup -h primary -D /data -S my_slot -X stream -R
```

The `-R` option is crucial - it:
1. Creates `standby.signal` file
2. Adds `primary_conninfo` to `postgresql.auto.conf`
3. Makes the backup ready to start as a standby

## Authentication Setup

On the primary, configure `pg_hba.conf`:

```
# Allow replication connections
host replication replicator standby-ip/32 scram-sha-256
```

Create the replication user:

```sql
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'secret';
```

## Code Examples

```undefined
# Full production backup command
pg_basebackup \
  --host=primary.example.com \
  --port=5432 \
  --username=replicator \
  --pgdata=/var/lib/postgresql/data \
  --wal-method=stream \
  --slot=standby_slot \
  --create-slot \
  --checkpoint=fast \
  --progress \
  --write-recovery-conf \
  --verbose
```


## Resources

- [pg_basebackup](https://www.postgresql.org/docs/current/app-pgbasebackup.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*