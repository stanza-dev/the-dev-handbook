---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-setup-configuration"
---

# Setting Up Action Cable

## Introduction
Before you can build real-time features, you need to configure Action Cable's adapter, allowed origins, and mount point. Rails 8.1 introduces Solid Cable as a first-party adapter option alongside Redis and PostgreSQL, giving you a database-backed alternative that requires no additional infrastructure.

## Key Concepts
- **Cable Adapter**: The pub/sub backend that Action Cable uses to broadcast messages between server processes. Options include `async` (single process, development only), `redis`, `postgresql`, and `solid_cable`.
- **Solid Cable**: A database-backed Action Cable adapter introduced in Rails 8. It stores messages in a dedicated database table, supporting MySQL, SQLite, and PostgreSQL without requiring a separate Redis instance.
- **Allowed Request Origins**: A security setting that restricts which domains can establish WebSocket connections, preventing cross-site WebSocket hijacking.
- **Worker Pool Size**: The number of threads Action Cable uses to process incoming messages. Each thread handles callbacks from subscriptions.

## Real World Context
In production, your Rails app typically runs multiple server processes (via Puma workers or multiple containers). The `async` adapter only works within a single process, so you need a shared pub/sub backend. Historically, Redis was the default choice. With Rails 8.1, Solid Cable lets you use your existing database as the pub/sub backend, eliminating an infrastructure dependency. This is especially useful for smaller applications or teams that want to minimize operational complexity.

## Deep Dive
Action Cable configuration lives in `config/cable.yml`. Here is a typical setup covering all environments:

```yaml
# config/cable.yml
development:
  adapter: async

test:
  adapter: test

production:
  adapter: solid_cable
```

The `async` adapter is perfect for development — it keeps messages in memory within the current process. For production, you have three main options:

**Solid Cable** (recommended for Rails 8.1):

Install Solid Cable with the built-in generator:

```bash
bin/rails solid_cable:install
```

This command sets up `config/cable.yml` with the `solid_cable` adapter and creates `db/cable_schema.rb` containing the table definition for message storage. Solid Cable supports MySQL, SQLite, and PostgreSQL. Messages are automatically trimmed after they expire, keeping the table small.

**Redis adapter**:

```yaml
production:
  adapter: redis
  url: <%= ENV.fetch("REDIS_URL", "redis://localhost:6379/1") %>
  channel_prefix: myapp_production
```

Redis is a high-performance option ideal for applications with very high message throughput. The `channel_prefix` prevents collisions if multiple apps share the same Redis instance.

**PostgreSQL adapter**:

```yaml
production:
  adapter: postgresql
```

The PostgreSQL adapter uses `NOTIFY/LISTEN` for pub/sub, leveraging your existing database without additional infrastructure. However, it has lower throughput than Redis for very high-volume scenarios.

Next, configure allowed origins to prevent unauthorized WebSocket connections:

```ruby
# config/environments/production.rb
Rails.application.configure do
  config.action_cable.allowed_request_origins = [
    "https://myapp.com",
    "https://www.myapp.com"
  ]

  # Or use a regex for flexibility
  # config.action_cable.allowed_request_origins = [/https:\/\/.*\.myapp\.com/]

  config.action_cable.worker_pool_size = 4
end
```

Finally, ensure the Action Cable JavaScript is loaded in your application. In Rails 8.1 with import maps:

```ruby
# config/importmap.rb
pin "@rails/actioncable", to: "actioncable.esm.js"
```

## Common Pitfalls
1. **Using the async adapter in production** — The `async` adapter only works within a single process. If you run multiple Puma workers or containers, messages will not reach subscribers on other processes. Always use `redis`, `postgresql`, or `solid_cable` in production.
2. **Forgetting allowed_request_origins** — In production, Action Cable rejects connections from origins not in the allowed list. If your WebSocket connections silently fail in production but work in development, check this setting first.
3. **Over-sizing the worker pool** — Each Action Cable worker is a thread. Setting the pool too high on a memory-constrained server can starve your main request threads. Start with 4 and increase based on monitoring.

## Best Practices
1. **Start with Solid Cable in Rails 8.1** — It requires no additional infrastructure and handles moderate message volumes well. Migrate to Redis only if monitoring reveals a bottleneck.
2. **Set `channel_prefix` when using Redis** — This prevents message collisions when multiple applications or environments share a Redis instance.
3. **Use environment-specific configuration** — Keep `async` for development, `test` for tests, and your production adapter separate. Never hardcode production credentials.

## Summary
- Action Cable supports four adapters: `async` (dev only), `redis`, `postgresql`, and `solid_cable` (new in Rails 8).
- Solid Cable is installed via `bin/rails solid_cable:install` and uses your database for pub/sub, supporting MySQL, SQLite, and PostgreSQL.
- Always configure `allowed_request_origins` in production to prevent cross-site WebSocket hijacking.
- The worker pool size controls how many threads process incoming WebSocket messages — start with 4.
- Use the `async` adapter for development and a shared backend (Solid Cable, Redis, or PostgreSQL) for production.

## Code Examples

**Cable configuration using Solid Cable for production — no Redis required, messages are stored in your database**

```yaml
# config/cable.yml
development:
  adapter: async

test:
  adapter: test

production:
  adapter: solid_cable
```

**Install Solid Cable — this sets up config/cable.yml and creates db/cable_schema.rb**

```bash
bin/rails solid_cable:install
```


## Resources

- [Action Cable Configuration](https://guides.rubyonrails.org/action_cable_overview.html#configuration) — Official guide covering adapter configuration and deployment options
- [Solid Cable GitHub Repository](https://github.com/rails/solid_cable) — Source code and documentation for the Solid Cable adapter

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*