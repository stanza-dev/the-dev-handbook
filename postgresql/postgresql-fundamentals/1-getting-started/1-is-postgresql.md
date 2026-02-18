---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-what-is-postgresql"
---

# What is PostgreSQL?

PostgreSQL (often called "Postgres") is the world's most advanced open-source relational database management system. It has earned a reputation for reliability, feature robustness, and performance over more than 35 years of active development.

## Why Choose PostgreSQL?

**Open Source & Free**: PostgreSQL is completely free with no licensing costs, even for commercial use.

**Standards Compliant**: PostgreSQL closely adheres to SQL standards while adding powerful extensions.

**Enterprise Features**: Support for complex queries, foreign keys, triggers, views, transactional integrity (ACID), and multiversion concurrency control (MVCC).

**Extensible**: Create custom data types, functions, operators, and even index types.

## Key Features in PostgreSQL 18

- **Asynchronous I/O (AIO)**: Improved performance for sequential scans, vacuums, and other operations
- **Virtual generated columns**: Computed values during read operations (now the default)
- **OAuth authentication**: Modern authentication support
- **Temporal constraints**: Constraints over ranges for PRIMARY KEY, UNIQUE, and FOREIGN KEY
- **Skip scan lookups**: Better utilization of multicolumn B-tree indexes

## The Client-Server Model

PostgreSQL uses a client-server architecture:

```
┌─────────────────┐         ┌─────────────────┐
│  Client (psql)  │ ──TCP── │  PostgreSQL     │
│  Application    │   or    │  Server         │
│  pgAdmin        │  Unix   │  (Database)     │
└─────────────────┘ Socket  └─────────────────┘
```

- **Server (postgres)**: Manages database files, accepts connections, performs database operations
- **Client**: Any application that connects to the server (psql, pgAdmin, your application)

## Who Uses PostgreSQL?

Major companies rely on PostgreSQL:
- Apple, Instagram, Spotify, Reddit, Twitch
- Financial institutions and government agencies
- NASA for mission-critical data

📖 [PostgreSQL Official Documentation](https://www.postgresql.org/docs/18/)

## Resources

- [PostgreSQL 18 Release Notes](https://www.postgresql.org/docs/18/release-18.html) — New features and improvements in PostgreSQL 18

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*