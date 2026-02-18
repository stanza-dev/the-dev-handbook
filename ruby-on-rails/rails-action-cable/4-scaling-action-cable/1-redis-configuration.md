---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-redis-configuration"
---

# Redis Configuration for Production

Redis is essential for Action Cable in production, enabling multiple Rails processes to share WebSocket messages.

## Why Redis?

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Rails 1    │     │   Rails 2    │     │   Rails 3    │
│  (Puma)      │     │  (Puma)      │     │  (Puma)      │
└──────┬───────┘     └──────┬───────┘     └──────┬───────┘
       │                    │                    │
       └────────────────────┼────────────────────┘
                            │
                     ┌──────┴───────┐
                     │    Redis     │
                     │  (pub/sub)   │
                     └──────────────┘
```

Without Redis, messages only reach users connected to the same process.

## Configuration

```yaml
# config/cable.yml
production:
  adapter: redis
  url: <%= ENV.fetch("REDIS_URL") { "redis://localhost:6379/1" } %>
  channel_prefix: myapp_production
```

## Redis URL Formats

```ruby
# Basic
redis://localhost:6379/1

# With password
redis://:password@localhost:6379/1

# With username and password (Redis 6+)
redis://username:password@localhost:6379/1

# SSL/TLS
rediss://localhost:6379/1

# Full example with all options
redis://:password@redis.example.com:6380/2
```

## Connection Pooling

```ruby
# config/initializers/action_cable.rb
ActionCable.server.config.cable = {
  adapter: 'redis',
  url: ENV['REDIS_URL'],
  channel_prefix: 'myapp_production'
}

# For high traffic, configure Redis connection pool
Rails.application.config.action_cable.worker_pool_size = 10
```

## Separate Redis Instance

For high-traffic apps, use a dedicated Redis for Action Cable:

```yaml
# config/cable.yml
production:
  adapter: redis
  url: <%= ENV.fetch("ACTION_CABLE_REDIS_URL") %>
  channel_prefix: myapp_cable
```

## Health Checks

```ruby
# config/routes.rb
get '/health/cable', to: proc {
  begin
    ActionCable.server.pubsub.redis_connection_for_subscriptions.ping
    [200, {}, ['OK']]
  rescue => e
    [503, {}, ["Redis connection failed: #{e.message}"]]
  end
}
```

## Monitoring Redis

```ruby
# Custom metrics
class CableMetrics
  def self.connection_count
    ActionCable.server.connections.count
  end

  def self.subscription_count
    ActionCable.server.connections.sum { |c| c.subscriptions.count }
  end
end
```

## PostgreSQL Alternative

For simpler setups without Redis:

```yaml
# config/cable.yml
production:
  adapter: postgresql
```

Uses PostgreSQL's NOTIFY/LISTEN feature. Good for:
- Smaller deployments
- Avoiding additional infrastructure
- Lower message volume

## Resources

- [Action Cable Configuration](https://guides.rubyonrails.org/action_cable_overview.html#deployment) — Production deployment configuration

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*