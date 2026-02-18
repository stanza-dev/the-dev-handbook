---
source_course: "postgresql-replication"
source_lesson: "why-replication-matters"
---

# Why Replication Matters

Database replication is the process of copying and maintaining database objects across multiple servers. In production environments, replication serves three critical purposes: **high availability** (keeping systems running during failures), **load balancing** (distributing read queries across servers), and **disaster recovery** (protecting against data loss).

## PostgreSQL Replication Types

PostgreSQL supports two fundamentally different replication approaches:

### Physical (Streaming) Replication

Physical replication copies the entire database cluster at the binary level. The standby server receives Write-Ahead Log (WAL) records from the primary and replays them to maintain an exact copy.

**Characteristics:**
- Replicates the entire cluster (all databases)
- Standby is an exact binary copy
- Very fast and efficient
- Supports synchronous mode for zero data loss
- Standby can serve read-only queries (Hot Standby)

### Logical Replication

Logical replication works at the row level, decoding WAL records into logical changes (INSERT, UPDATE, DELETE) and applying them selectively.

**Characteristics:**
- Replicates specific tables, not entire cluster
- Subscriber can have different indexes, triggers
- Supports cross-version replication
- Enables selective data replication with row/column filters
- Subscriber can be read-write

## When to Use Each Type

| Scenario | Recommended Type |
|----------|------------------|
| Full database failover | Physical |
| Data warehouse feeding | Logical |
| Horizontal read scaling | Physical |
| Selective table sync | Logical |
| Cross-version upgrade | Logical |
| Zero data loss requirement | Physical (synchronous) |

## Resources

- [High Availability and Replication](https://www.postgresql.org/docs/current/high-availability.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*