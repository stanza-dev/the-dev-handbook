---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-what-is-redis"
---

# What is Redis?

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

Redis 8 introduces significant improvements:

- **Hash field expiration**: Set TTL on individual hash fields
- **Enhanced performance**: Optimized memory management
- **Vector search**: Built-in support for AI/ML applications
- **Improved clustering**: Better stability and performance

## Who Uses Redis?

Redis powers some of the world's largest applications:
- Twitter (timeline caching)
- GitHub (job queues)
- Stack Overflow (caching layer)
- Snapchat (messaging)
- Pinterest (recommendations)

📖 [Official Redis Documentation](https://redis.io/docs/latest/)

## Resources

- [Redis Introduction](https://redis.io/docs/latest/develop/get-started/) — Official getting started guide for Redis

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*