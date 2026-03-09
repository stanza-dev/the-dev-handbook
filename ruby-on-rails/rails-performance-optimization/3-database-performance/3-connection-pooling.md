---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-connection-pooling"
---

# Database Connection Management

## Introduction
Every database query requires a connection. Rails manages a pool of reusable connections, but misconfiguration can lead to timeouts, connection exhaustion, and degraded performance under load.

## Key Concepts
- **Connection Pool**: A fixed set of reusable database connections shared across threads in a Rails process.
- **Pool Size**: The maximum number of concurrent connections a single Rails process can hold. Must match your thread count.
- **PgBouncer**: An external connection pooler that sits between Rails and PostgreSQL, multiplexing many application connections onto fewer database connections.

## Real World Context
A Puma server with 3 workers and 5 threads each needs 15 database connections minimum. With Sidekiq running 25 threads, you need 25 more. Without proper pooling, you'll hit `ActiveRecord::ConnectionTimeoutError` under load.

## Deep Dive

### Pool Configuration

```yaml
# config/database.yml
production:
  adapter: postgresql
  pool: <%= ENV.fetch('RAILS_MAX_THREADS', 5) %>
  url: <%= ENV['DATABASE_URL'] %>
  prepared_statements: true
```

### Puma Thread/Pool Alignment

```ruby
# config/puma.rb
workers ENV.fetch('WEB_CONCURRENCY', 2)
threads_count = ENV.fetch('RAILS_MAX_THREADS', 5)
threads threads_count, threads_count

preload_app!

on_worker_boot do
  ActiveRecord::Base.establish_connection
end
```

Each Puma worker forks a new process. `preload_app!` loads the app once, then each worker re-establishes its own connection pool.

### Connection Checkout Timeout

```yaml
production:
  pool: 5
  checkout_timeout: 5  # Wait 5 seconds for a connection (default)
```

If all connections are busy for longer than `checkout_timeout`, Rails raises `ConnectionTimeoutError`.

### Using PgBouncer

When running many processes (web + workers + jobs), PgBouncer reduces the actual database connection count:

```yaml
# config/database.yml (with PgBouncer)
production:
  url: <%= ENV['DATABASE_URL'] %>  # Points to PgBouncer
  prepared_statements: false  # Required with PgBouncer in transaction mode
  advisory_locks: false       # Required with PgBouncer
```

## Common Pitfalls
1. **Pool size smaller than thread count** — If your pool is 5 but Puma runs 10 threads, 5 threads will wait for connections on every request.
2. **Forgetting to re-establish connections after fork** — Puma workers share the parent's connections after forking. Always call `establish_connection` in `on_worker_boot`.

## Best Practices
1. **Match pool size to thread count** — Set `pool` equal to `RAILS_MAX_THREADS` for both Puma and Sidekiq.
2. **Monitor connection usage** — Track active connections with `ActiveRecord::Base.connection_pool.stat` to detect exhaustion before it causes errors.

## Summary
- Set database pool size equal to your thread count per process.
- Re-establish connections after Puma worker forks with `on_worker_boot`.
- Use PgBouncer when running many processes to reduce database connection count.
- Monitor pool stats to detect exhaustion early.

## Code Examples

**Inspecting connection pool stats — busy + waiting reveals if your pool is too small**

```ruby
# Check connection pool stats in Rails console
stats = ActiveRecord::Base.connection_pool.stat
puts stats
# => { size: 5, connections: 3, busy: 1, dead: 0,
#      idle: 2, waiting: 0, checkout_timeout: 5.0 }

# size: pool max, busy: in-use, waiting: threads waiting for a connection
```


## Resources

- [Configuring Rails Applications — Database Pooling](https://guides.rubyonrails.org/configuring.html#database-pooling) — Official Rails guide on configuring database connection pools

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*