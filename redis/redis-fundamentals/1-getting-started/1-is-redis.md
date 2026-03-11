---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-what-is-redis"
---

# What is Redis?

## Introduction

Redis (Remote Dictionary Server) is the world's most popular in-memory data store, used as a database, cache, message broker, and streaming engine. In this lesson you will learn what makes Redis unique, when to use it, and what changed in Redis 8.

## Key Concepts

- **In-memory storage**: All data lives in RAM, enabling sub-millisecond read/write latency.
- **Data structures**: Redis is not a plain key-value store — it natively supports strings, hashes, lists, sets, sorted sets, streams, JSON, and more.
- **Persistence**: Optional disk persistence (RDB/AOF) means data survives restarts despite being in-memory.
- **Single-threaded command execution**: Redis processes commands sequentially on one thread, avoiding lock contention. Redis 8 adds configurable I/O threading for network operations, but command processing remains single-threaded.

## Real World Context

Nearly every large-scale web application uses Redis. Twitter caches timelines, GitHub processes background jobs, and Snapchat routes messages — all through Redis. Whenever you need microsecond latency for reads/writes or need to share state across application servers, Redis is the go-to solution.

## Deep Dive

Redis (Remote Dictionary Server) is an open-source, in-memory data structure store that can be used as a database, cache, message broker, and streaming engine. It's known for its exceptional speed, typically serving millions of requests per second with sub-millisecond latency.

## Why Choose Redis?

**Blazing Fast Performance**: Redis stores data in memory, making read and write operations incredibly fast. Most operations complete in microseconds.

**Rich Data Structures**: Unlike simple key-value stores, Redis supports complex data types:
- Strings
- Hashes (like objects)
- Lists (linked lists)
- Sets (unique values)
- Sorted Sets (ranked data)
- Streams (event logs)
- And more...

**Versatile Use Cases**: Redis excels at:
- Caching (session storage, page caching)
- Real-time analytics (leaderboards, counters)
- Message queues (Pub/Sub, Streams)
- Rate limiting
- Geospatial indexing

## How Redis Works

Redis uses a client-server architecture:

```
┌─────────────────┐         ┌─────────────────┐
│  Your App       │         │  Redis Server   │
│  (Client)       │ ──TCP── │  (In-Memory)    │
│                 │         │                 │
└─────────────────┘         └─────────────────┘
```

1. **Your application** connects to Redis as a client
2. **Redis server** stores all data in RAM for speed
3. **Optional persistence** saves data to disk for durability

## Redis 8 Key Features

Redis 8 is a major release with significant improvements:

- **Built-in data modules**: JSON, Search and Query, Time Series, and Probabilistic data structures are now built-in — no separate modules required
- **New hash commands**: HGETEX, HSETEX, and HGETDEL combine read/write with expiration or deletion atomically
- **Hash field expiration**: Set TTL on individual hash fields (introduced in 7.4, enhanced in 8)
- **Vector search (beta)**: Built-in vector similarity search for AI/ML applications (beta in 8.0)
- **Performance gains**: Up to 87% lower command latency, 35% memory savings for replicas
- **I/O threading**: New configurable multi-threaded I/O via the `io-threads` parameter for higher throughput on multi-core systems

## Who Uses Redis?

Redis powers some of the world's largest applications:
- Twitter (timeline caching)
- GitHub (job queues)
- Stack Overflow (caching layer)
- Snapchat (messaging)
- Pinterest (recommendations)

📖 [Official Redis Documentation](https://redis.io/docs/latest/)

## Common Pitfalls

1. **Treating Redis as a durable primary database without persistence** — By default Redis can lose data on restart. Always configure persistence if durability matters.
2. **Storing too much data** — Redis is limited by available RAM. Use it for hot data and let disk-based databases handle cold storage.

## Best Practices

1. **Use the right data structure** — Don't serialize everything into strings. Hashes, sets, and sorted sets are purpose-built for common patterns.
2. **Set maxmemory and an eviction policy** — Prevent Redis from consuming all system memory in production.

## Summary

- Redis is an in-memory data structure store with sub-millisecond latency.
- It supports rich data types beyond simple key-value pairs.
- Redis 8 bundles JSON, Search, Time Series, and Probabilistic data structures built-in.
- Persistence options (RDB/AOF) provide durability when needed.
- Redis is used by Twitter, GitHub, Snapchat, and millions of other applications.

## Code Examples

**Your first Redis interaction — PING tests the connection, SET stores a value, GET retrieves it**

```bash
# Connect to Redis and try basic operations
redis-cli
127.0.0.1:6379> PING
PONG
127.0.0.1:6379> SET hello "world"
OK
127.0.0.1:6379> GET hello
"world"
```


## Resources

- [Redis Introduction](https://redis.io/docs/latest/develop/get-started/) — Official getting started guide for Redis

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*