---
source_course: "postgresql-replication"
source_lesson: "postgresql-replication-pg-basebackup"
---

# Creating Base Backups with pg_basebackup

## Introduction

`pg_basebackup` is the standard PostgreSQL tool for creating physical backups and initializing standby servers. It creates a consistent copy of the entire database cluster over a replication connection.

This lesson covers backup formats, essential flags, authentication setup, and the critical `-R` option for standby initialization.

## Key Concepts

- **pg_basebackup**: A utility that takes a base backup of a running PostgreSQL server, producing a consistent snapshot of the entire cluster.
- **Plain format (-Fp)**: Writes the backup as a regular directory structure, ready to start as a PostgreSQL data directory.
- **Tar format (-Ft)**: Writes the backup as tar archives, which can be compressed and are more portable.
- **-R (--write-recovery-conf)**: Creates `standby.signal` and writes `primary_conninfo` to `postgresql.auto.conf`, making the backup ready to start as a standby.

## Real World Context

When provisioning a new read replica for a growing e-commerce platform, you run `pg_basebackup` with the `-R` flag against the primary. This creates a complete copy of the cluster data and automatically configures the new server as a standby. The entire process can complete while the primary continues serving traffic, with no downtime required.

## Deep Dive

The basic usage creates a backup in a target directory:

```bash
# Simple backup to directory
pg_basebackup -h primary-host -D /backup/base -P -X stream

# Options explained:
# -h: Primary server hostname
# -D: Destination directory
# -P: Show progress
# -X stream: Stream WAL during backup (recommended)
```

The `-X stream` option is important because it captures WAL generated during the backup, ensuring consistency without relying on WAL archiving.

You can choose between plain and tar formats:

```bash
# Plain format (default) - ready to start as a data directory
pg_basebackup -D /backup/base -Fp

# Tar format - compressed, portable
pg_basebackup -D /backup -Ft -z

# With zstd compression (PostgreSQL 15+)
pg_basebackup -D /backup -Ft --compress=zstd:level=5
```

Tar format is preferred for archival storage, while plain format is used when you want to start the backup directly.

To create a standby-ready backup, use the `-R` flag:

```bash
# Create backup with standby signal
pg_basebackup -h primary -D /var/lib/postgresql/data \
  -X stream \
  -P \
  -R    # Creates standby.signal and configures primary_conninfo

# Using a replication slot (prevents WAL removal during backup)
pg_basebackup -h primary -D /data -S my_slot -X stream -R
```

The `-R` option performs three actions: it creates the `standby.signal` file, adds `primary_conninfo` to `postgresql.auto.conf`, and makes the backup ready to start as a streaming standby.

On the primary, configure authentication for replication:

```
# pg_hba.conf - Allow replication connections
host replication replicator standby-ip/32 scram-sha-256
```

Create the replication user:

```sql
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'secure_password';
```

The replication user needs only the REPLICATION privilege, not superuser access.

## Common Pitfalls

- **Not using -X stream**: Without it, the backup relies on WAL archiving to be consistent. If archiving fails or is not configured, the backup may be unusable.
- **Running out of disk during backup**: The backup is as large as the database cluster. Verify sufficient disk space before starting.
- **Forgetting pg_hba.conf entries**: The replication connection uses a special `replication` pseudo-database in pg_hba.conf, not the actual database name.

## Best Practices

- Always use `-X stream` to include WAL in the backup for self-contained consistency.
- Use a dedicated replication slot with `-S` for long-running backups to prevent WAL recycling.
- Use `--checkpoint=fast` to start the backup immediately rather than waiting for the next regular checkpoint.

## Summary

- `pg_basebackup` creates a consistent physical copy of the entire database cluster.
- Plain format is ready to start as a data directory; tar format is compressed and portable.
- The `-R` flag configures the backup as a streaming standby automatically.
- Always use `-X stream` to include WAL segments generated during the backup.
- Configure `pg_hba.conf` and create a dedicated replication user for authentication.

## Code Examples

**Production pg_basebackup Command**

```bash
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