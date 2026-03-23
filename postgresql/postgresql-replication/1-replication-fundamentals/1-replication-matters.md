---
source_course: "postgresql-replication"
source_lesson: "postgresql-replication-why-replication-matters"
---

# Why Replication Matters

## Introduction

Database replication is the process of copying and maintaining database objects across multiple servers. In production environments, replication serves three critical purposes: **high availability** (keeping systems running during failures), **load balancing** (distributing read queries across servers), and **disaster recovery** (protecting against data loss).

This lesson provides a foundational understanding of the two primary replication approaches PostgreSQL offers and when to use each one.

## Key Concepts

- **Physical (Streaming) Replication**: Copies the entire database cluster at the binary level by shipping Write-Ahead Log (WAL) records from primary to standby servers.
- **Logical Replication**: Works at the row level, decoding WAL records into logical changes (INSERT, UPDATE, DELETE) and applying them selectively to subscribers.
- **High Availability**: The ability of a database system to remain operational during hardware or software failures by failing over to a replica.
- **Write-Ahead Log (WAL)**: PostgreSQL's mechanism for recording all changes before they are applied, forming the backbone of both replication types.

## Real World Context

Consider a SaaS application serving thousands of users. If your single database server fails, every user loses access. With streaming replication, a standby server can take over within seconds. For a data warehouse that only needs specific tables from the production database, logical replication lets you selectively sync just the tables you need without copying the entire cluster.

## Deep Dive

PostgreSQL supports two fundamentally different replication approaches, each with distinct characteristics and use cases.

### Physical (Streaming) Replication

Physical replication copies the entire database cluster at the binary level. The standby server receives WAL records from the primary and replays them to maintain an exact copy.

The key characteristics of physical replication are:

- Replicates the entire cluster (all databases)
- Standby is an exact binary copy
- Very fast and efficient
- Supports synchronous mode for zero data loss
- Standby can serve read-only queries (Hot Standby)

### Logical Replication

Logical replication works at the row level, decoding WAL records into logical changes and applying them selectively.

The key characteristics of logical replication are:

- Replicates specific tables, not entire cluster
- Subscriber can have different indexes, triggers
- Supports cross-version replication
- Enables selective data replication with row/column filters
- Subscriber can be read-write

The following table summarizes when to use each type:

| Scenario | Recommended Type |
|----------|------------------|
| Full database failover | Physical |
| Data warehouse feeding | Logical |
| Horizontal read scaling | Physical |
| Selective table sync | Logical |
| Cross-version upgrade | Logical |
| Zero data loss requirement | Physical (synchronous) |

## Common Pitfalls

- **Choosing logical replication for full failover**: Logical replication does not replicate DDL, sequences, or large objects, making it unsuitable as a sole failover mechanism.
- **Forgetting that physical replication copies everything**: You cannot exclude specific databases or tables from physical replication; it is all-or-nothing at the cluster level.
- **Overlooking WAL level requirements**: Physical replication requires `wal_level = replica`, while logical replication requires `wal_level = logical`. Setting the wrong level means replication will not work.

## Best Practices

- Use physical replication as your first line of defense for high availability and disaster recovery.
- Use logical replication for selective data distribution, cross-version migrations, or feeding analytics systems.
- Always plan for both read scaling and failover when designing your replication topology.

## Summary

- PostgreSQL offers two replication types: physical (binary-level, entire cluster) and logical (row-level, selective tables).
- Physical replication is ideal for high availability and read scaling; logical replication excels at selective data distribution.
- Choosing the right replication type depends on your use case: failover, scaling, data distribution, or migration.
- Both types rely on WAL as the underlying mechanism for tracking and shipping changes.

## Resources

- [High Availability and Replication](https://www.postgresql.org/docs/current/high-availability.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*