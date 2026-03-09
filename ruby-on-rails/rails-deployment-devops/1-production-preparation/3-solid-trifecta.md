---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-solid-trifecta"
---

# The Solid Trifecta

## Introduction
Rails 8 introduces the Solid Trifecta: Solid Cache, Solid Queue, and Solid Cable. These three database-backed libraries replace Redis for caching, background jobs, and WebSocket communication, dramatically simplifying Rails infrastructure.

## Key Concepts
- **Solid Cache**: Database-backed cache store that replaces Redis for caching. Handles most workloads efficiently.
- **Solid Queue**: Database-backed job queue that replaces Sidekiq/Redis for background processing. Supports priorities, concurrency limits, and delayed jobs.
- **Solid Cable**: Database-backed Action Cable adapter that replaces Redis for WebSocket pub/sub.

## Real World Context
Before Rails 8, a typical production deployment required PostgreSQL + Redis + Sidekiq. Now you just need PostgreSQL. This reduces infrastructure cost, operational complexity, and the number of things that can fail.

## Deep Dive

### Solid Cache

```ruby
# config/environments/production.rb
config.cache_store = :solid_cache_store
```

Solid Cache stores cached values in a database table. It's the default in Rails 8 and handles most caching workloads without Redis.

### Solid Queue

```ruby
# config/environments/production.rb
config.active_job.queue_adapter = :solid_queue
config.solid_queue.connects_to = { database: { writing: :queue } }
```

Run embedded in Puma (simplest deployment):

```bash
SOLID_QUEUE_IN_PUMA=true
```

Or as a separate process:

```bash
bundle exec rake solid_queue:start
```

### Solid Cable

```yaml
# config/cable.yml
production:
  adapter: solid_cable
  connects_to:
    database:
      writing: cable
  polling_interval: 0.1.seconds
```

### Database Setup for All Three

```yaml
# config/database.yml
production:
  primary:
    url: <%= ENV['DATABASE_URL'] %>
  queue:
    url: <%= ENV.fetch('QUEUE_DATABASE_URL') { ENV['DATABASE_URL'] } %>
    migrations_paths: db/queue_migrate
  cache:
    url: <%= ENV.fetch('CACHE_DATABASE_URL') { ENV['DATABASE_URL'] } %>
    migrations_paths: db/cache_migrate
  cable:
    url: <%= ENV.fetch('CABLE_DATABASE_URL') { ENV['DATABASE_URL'] } %>
    migrations_paths: db/cable_migrate
```

### When to Use Redis Instead

Redis is still valuable for:
- Very high-throughput caching (millions of reads/sec)
- Sub-millisecond latency requirements
- Large-scale pub/sub (thousands of WebSocket connections)

```ruby
# Optional: use Redis for cache if needed
config.cache_store = :redis_cache_store, {
  url: ENV['REDIS_URL'],
  expires_in: 1.day
}
```

## Common Pitfalls
1. **Using a single database for everything** — For high-traffic apps, separate the queue, cache, and cable databases to avoid lock contention with your primary data.
2. **Adding Redis by default** — Start with the Solid Trifecta. Only add Redis if profiling shows you need it.

## Best Practices
1. **Start simple** — Use the Solid Trifecta with a single database. Split into separate databases only when traffic demands it.
2. **Monitor queue latency** — If Solid Queue jobs start backing up, consider running a separate worker process instead of embedded mode.

## Summary
- The Solid Trifecta (Cache, Queue, Cable) eliminates Redis for most Rails apps.
- All three are database-backed and included by default in Rails 8.
- Solid Queue can run embedded in Puma or as a separate process.
- Use separate databases for queue/cache/cable in high-traffic apps.
- Redis remains an option for extreme throughput requirements.

## Code Examples

**The Solid Trifecta configuration — Solid Cache, Solid Queue, and Solid Cable replace Redis entirely**

```ruby
# Rails 8 production — no Redis needed!
# config/environments/production.rb
config.cache_store = :solid_cache_store          # Database cache
config.active_job.queue_adapter = :solid_queue    # Database jobs

# config/cable.yml
# production:
#   adapter: solid_cable                         # Database WebSockets

# Just PostgreSQL — that's the entire infrastructure
```


## Resources

- [Rails 8 Release Notes](https://guides.rubyonrails.org/8_0_release_notes.html) — Official Rails 8 release notes covering the Solid Trifecta and other major features

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*