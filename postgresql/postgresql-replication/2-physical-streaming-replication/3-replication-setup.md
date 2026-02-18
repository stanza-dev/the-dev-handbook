---
source_course: "postgresql-replication"
source_lesson: "streaming-replication-setup"
---

# Configuring Streaming Replication

Streaming replication sends WAL records directly from primary to standby over a TCP connection, providing near-real-time replication.

## Primary Server Configuration

```sql
-- postgresql.conf on primary
wal_level = replica
max_wal_senders = 10
max_replication_slots = 10
hot_standby = on

-- Recommended for reliability
synchronous_commit = on
wal_keep_size = 1GB
```

## Replication Slots

Replication slots prevent the primary from removing WAL files that standby still needs:

```sql
-- Create a physical replication slot
SELECT pg_create_physical_replication_slot('standby_slot');

-- View existing slots
SELECT slot_name, slot_type, active, restart_lsn
FROM pg_replication_slots;

-- Drop unused slot (WARNING: can cause WAL bloat if forgotten)
SELECT pg_drop_replication_slot('old_slot');
```

## Standby Configuration

Create `standby.signal` in data directory and configure:

```sql
-- postgresql.auto.conf or postgresql.conf on standby
primary_conninfo = 'host=primary port=5432 user=replicator password=secret'
primary_slot_name = 'standby_slot'
hot_standby = on
```

## Monitoring Replication

```sql
-- On primary: view connected standbys
SELECT client_addr, state, sent_lsn, write_lsn, flush_lsn, replay_lsn,
       pg_wal_lsn_diff(sent_lsn, replay_lsn) AS lag_bytes
FROM pg_stat_replication;

-- On standby: view replication status
SELECT pg_is_in_recovery();  -- Should return true
SELECT pg_last_wal_receive_lsn();
SELECT pg_last_wal_replay_lsn();

-- Calculate lag in seconds
SELECT EXTRACT(EPOCH FROM (now() - pg_last_xact_replay_timestamp()));
```

## Code Examples

```undefined
-- On PRIMARY: Create replication user and slot
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'secure_password';
SELECT pg_create_physical_replication_slot('standby1_slot');

-- On STANDBY: After pg_basebackup -R, verify connection
-- Check postgresql.auto.conf contains:
-- primary_conninfo = 'host=primary port=5432 user=replicator password=...'
-- primary_slot_name = 'standby1_slot'
```


## Resources

- [Log-Shipping Standby Servers](https://www.postgresql.org/docs/current/warm-standby.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*