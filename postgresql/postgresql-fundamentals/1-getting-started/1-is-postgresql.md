---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-what-is-postgresql"
---

# What is PostgreSQL?

## Introduction
PostgreSQL (often called "Postgres") is the world's most advanced open-source relational database management system. Over more than 35 years of active development, it has earned a reputation for reliability, feature robustness, and performance. In this lesson, you will learn what PostgreSQL is, why it matters, and how its client-server architecture works.

## Key Concepts
- **Relational Database Management System (RDBMS)**: Software that stores data in tables with rows and columns, and enforces relationships between tables.
- **ACID Compliance**: Guarantees that database transactions are Atomic, Consistent, Isolated, and Durable — meaning your data stays correct even if the system crashes.
- **MVCC (Multiversion Concurrency Control)**: A technique that allows multiple users to read and write simultaneously without locking each other out.
- **Client-Server Architecture**: The database runs as a server process, and applications connect to it as clients over a network or local socket.

## Real World Context
Companies like Apple, Instagram, Spotify, Reddit, and Twitch rely on PostgreSQL for production workloads. Financial institutions and government agencies choose it for mission-critical data because of its stability and standards compliance. Understanding PostgreSQL opens doors to roles at virtually any tech company.

## Deep Dive

### Why Choose PostgreSQL?

PostgreSQL stands out from other databases for several reasons. Here is a summary of its core strengths:

- **Open Source & Free**: PostgreSQL is completely free with no licensing costs, even for commercial use.
- **Standards Compliant**: PostgreSQL closely adheres to SQL standards while adding powerful extensions.
- **Enterprise Features**: Support for complex queries, foreign keys, triggers, views, transactional integrity (ACID), and multiversion concurrency control (MVCC).
- **Extensible**: Create custom data types, functions, operators, and even index types.

### Key Features in PostgreSQL 18

PostgreSQL 18 introduced several important features that improve performance and developer experience:

- **Asynchronous I/O (AIO)**: Improved performance for sequential scans, vacuums, and other operations
- **Virtual generated columns**: Computed values during read operations (now the default)
- **OAuth authentication**: Modern authentication support
- **Temporal constraints**: Constraints over ranges for PRIMARY KEY, UNIQUE, and FOREIGN KEY
- **Skip scan lookups**: Better utilization of multicolumn B-tree indexes

These features make PostgreSQL 18 faster and more flexible than previous versions.

### The Client-Server Model

PostgreSQL uses a client-server architecture where the server manages data and clients connect to it. The diagram below illustrates this relationship:

```
┌─────────────────┐         ┌─────────────────┐
│  Client (psql)  │ ──TCP── │  PostgreSQL     │
│  Application    │   or    │  Server         │
│  pgAdmin        │  Unix   │  (Database)     │
└─────────────────┘ Socket  └─────────────────┘
```

The **Server (postgres)** manages database files, accepts connections, and performs database operations. The **Client** is any application that connects to the server — this could be psql (the command-line tool), pgAdmin (a GUI), or your application code.

## Common Pitfalls
1. **Confusing PostgreSQL with MySQL syntax** — While both are SQL databases, they have differences in data types, functions, and features. PostgreSQL uses `SERIAL` instead of `AUTO_INCREMENT`, and `ILIKE` instead of case-insensitive `LIKE`.
2. **Assuming PostgreSQL is slow because it is open source** — PostgreSQL performs on par with or better than commercial databases for most workloads. Its query planner is one of the most sophisticated in any RDBMS.

## Best Practices
1. **Always use the latest stable version** — Each release brings performance improvements, security patches, and new features. PostgreSQL 18 is a significant step forward.
2. **Read the official documentation** — PostgreSQL has some of the best documentation of any open-source project. The docs at postgresql.org are comprehensive and well-organized.

## Summary
- PostgreSQL is a free, open-source relational database with over 35 years of development.
- It supports ACID transactions, MVCC, and is highly extensible.
- It uses a client-server architecture where applications connect to a central database server.
- Major companies rely on PostgreSQL for production workloads.
- PostgreSQL 18 adds features like asynchronous I/O and virtual generated columns.

## Code Examples

**Verify which PostgreSQL version is running on your server using the built-in version() function**

```sql
-- Check your PostgreSQL version
SELECT version();
-- Output: PostgreSQL 18.3 on x86_64-pc-linux-gnu, compiled by gcc...
```


## Resources

- [PostgreSQL 18 Release Notes](https://www.postgresql.org/docs/18/release-18.html) — New features and improvements in PostgreSQL 18

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*