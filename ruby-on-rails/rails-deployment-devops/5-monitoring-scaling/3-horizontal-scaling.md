---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-horizontal-scaling"
---

# Horizontal Scaling

Scale horizontally by adding more servers rather than bigger servers.

## Scaling with Kamal

```yaml
# config/deploy.yml
servers:
  web:
    hosts:
      - web1.example.com
      - web2.example.com
      - web3.example.com  # Add more hosts
    options:
      memory: 2g
      cpus: 2
```

```bash
# Add a new server
kamal server add web4.example.com
kamal deploy
```

## Load Balancer Configuration

```yaml
# Traefik configuration in config/deploy.yml
traefik:
  args:
    # Health check configuration
    entryPoints.web.address: ':80'
    # Load balancing strategy
    api.insecure: 'true'
```

## Database Connection Pooling

```yaml
# config/database.yml
production:
  adapter: postgresql
  pool: <%= ENV.fetch('RAILS_MAX_THREADS', 5) %>
  
  # With PgBouncer
  prepared_statements: false
  advisory_locks: false
```

## Redis Connection Pooling

```ruby
# config/initializers/redis.rb
REDIS_POOL = ConnectionPool.new(size: 10) do
  Redis.new(url: ENV['REDIS_URL'])
end

# Usage
REDIS_POOL.with do |redis|
  redis.get('key')
end
```

## Caching for Scale

```ruby
# config/environments/production.rb
config.cache_store = :redis_cache_store, {
  url: ENV['REDIS_URL'],
  pool_size: ENV.fetch('RAILS_MAX_THREADS', 5),
  expires_in: 1.day
}
```

## Auto-Scaling

```yaml
# Example: AWS Auto Scaling policy
auto_scaling:
  min_instances: 2
  max_instances: 10
  target_cpu_utilization: 70
  scale_out_cooldown: 300
  scale_in_cooldown: 300
```

## Performance Tuning

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

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*