---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-horizontal-scaling"
---

# Horizontal Scaling Strategies

## Introduction
Horizontal scaling adds more servers to handle increased traffic, rather than upgrading to bigger servers. Kamal makes this as simple as adding host IPs to your deploy.yml.

## Key Concepts
- **Horizontal Scaling**: Adding more server instances to distribute load.
- **Connection Pooling**: Managing a fixed set of reusable database connections per process to prevent connection exhaustion.
- **Puma Tuning**: Configuring Puma's workers (processes) and threads to maximize throughput for your workload.

## Real World Context
Black Friday traffic is 10x normal. With Kamal, you add 5 new server IPs to deploy.yml, run `kamal deploy`, and you're handling the load. After the rush, remove the servers.

## Deep Dive

### Adding Servers with Kamal

```yaml
# config/deploy.yml
servers:
  web:
    hosts:
      - web1.example.com
      - web2.example.com
      - web3.example.com  # Add more as needed
```

```bash
kamal deploy  # Deploys to all hosts
```

### Puma Tuning

```ruby
# config/puma.rb
workers ENV.fetch('WEB_CONCURRENCY', 2)
threads_count = ENV.fetch('RAILS_MAX_THREADS', 5)
threads threads_count, threads_count

preload_app!
# Note: on_worker_boot with establish_connection is no longer needed
# in Rails 7.1+ — Active Record automatically re-establishes
# connections after fork when using preload_app!
```

### Database Connection Pooling

```yaml
# config/database.yml
production:
  adapter: postgresql
  pool: <%= ENV.fetch('RAILS_MAX_THREADS', 5) %>
  url: <%= ENV['DATABASE_URL'] %>
```

Total connections = workers × threads × number_of_servers.

### Caching for Scale

```ruby
# config/environments/production.rb
# Solid Cache (default — scales with your database)
config.cache_store = :solid_cache_store

# Or Redis for very high-throughput caching
config.cache_store = :redis_cache_store, {
  url: ENV['REDIS_URL'],
  pool_size: ENV.fetch('RAILS_MAX_THREADS', 5)
}
```

## Common Pitfalls
1. **Not matching pool size to threads** — Each thread needs its own database connection. Pool must be >= thread count.
2. **Adding unnecessary `on_worker_boot` blocks** — In Rails 7.1+, Active Record automatically re-establishes connections after fork. Adding manual `establish_connection` calls is redundant.

## Best Practices
1. **Start with 2 workers, 5 threads** — This is Puma's default and works well for most Rails apps. Profile before tuning.
2. **Use PgBouncer for many servers** — With 10+ servers, PgBouncer multiplexes connections to stay within PostgreSQL's limits.

## Summary
- Add server IPs to deploy.yml and run `kamal deploy` to scale horizontally.
- Match database pool size to your Puma thread count.
- Use `preload_app!` for efficient process forking (Rails 7.1+ handles connection re-establishment automatically).
- Monitor total database connections across all servers.

## Code Examples

**Puma configuration for production — workers control processes, threads control concurrency per process**

```ruby
# config/puma.rb — production tuning
workers ENV.fetch('WEB_CONCURRENCY', 2)   # Processes
threads_count = ENV.fetch('RAILS_MAX_THREADS', 5)
threads threads_count, threads_count       # Threads per worker

preload_app!  # Load app once, fork workers
# Rails 7.1+ automatically re-establishes DB connections after fork

# Total capacity: 2 workers × 5 threads = 10 concurrent requests
# Total DB connections: 2 × 5 = 10
```


## Resources

- [Puma Documentation](https://github.com/puma/puma) — Official Puma web server documentation covering workers, threads, and production tuning

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*