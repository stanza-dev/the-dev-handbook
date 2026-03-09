---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-production-configuration"
---

# Production Configuration

## Introduction
Production requires specific configuration for security, performance, and reliability. Rails 8 introduces sensible defaults including Solid Cache, Solid Queue, and Solid Cable — eliminating the need for Redis in most applications.

## Key Concepts
- **Eager Loading**: Loading all application code at boot time (`config.eager_load = true`) to avoid lazy-loading race conditions in multi-threaded servers.
- **Reloading Disabled**: In production, code is loaded once and never reloaded (`config.enable_reloading = false`).
- **Credentials**: Encrypted files that store secrets securely in version control, decrypted at runtime with a master key.

## Real World Context
A misconfigured production app can expose stack traces to users, serve unencrypted traffic, or run out of database connections. Getting production configuration right is the foundation of a secure, performant deployment.

## Deep Dive

### Rails 8 Production Configuration

```ruby
# config/environments/production.rb
Rails.application.configure do
  # Code is loaded once, never reloaded
  config.enable_reloading = false
  config.eager_load = true

  # Don't show detailed errors to users
  config.consider_all_requests_local = false

  # Caching with Solid Cache (Rails 8 default — database-backed)
  config.cache_store = :solid_cache_store

  # Force SSL
  config.force_ssl = true

  # Logging
  config.log_level = :info
  config.log_tags = [:request_id]

  # Background jobs with Solid Queue (Rails 8 default)
  config.active_job.queue_adapter = :solid_queue
  config.solid_queue.connects_to = { database: { writing: :queue } }

  # Email
  config.action_mailer.delivery_method = :smtp
  config.action_mailer.smtp_settings = {
    address: ENV['SMTP_HOST'],
    port: ENV['SMTP_PORT'],
    user_name: ENV['SMTP_USER'],
    password: ENV['SMTP_PASSWORD'],
    authentication: :plain,
    enable_starttls_auto: true
  }
end
```

### Credentials Management

```bash
RAILS_MASTER_KEY=xxx bin/rails credentials:edit --environment production
```

### Database Configuration

```yaml
# config/database.yml
production:
  primary:
    adapter: postgresql
    encoding: unicode
    pool: <%= ENV.fetch('RAILS_MAX_THREADS', 5) %>
    url: <%= ENV['DATABASE_URL'] %>
  queue:
    adapter: postgresql
    url: <%= ENV.fetch('QUEUE_DATABASE_URL') { ENV['DATABASE_URL'] } %>
    migrations_paths: db/queue_migrate
  cache:
    adapter: postgresql
    url: <%= ENV.fetch('CACHE_DATABASE_URL') { ENV['DATABASE_URL'] } %>
    migrations_paths: db/cache_migrate
```

### Required Environment Variables

```bash
RAILS_ENV=production
RAILS_MASTER_KEY=your_master_key
DATABASE_URL=postgres://user:pass@host:5432/app
SECRET_KEY_BASE=generated_secret
```

## Common Pitfalls
1. **Using `config.cache_classes`** — This setting is deprecated in Rails 8. Use `config.enable_reloading = false` instead.
2. **Forgetting to set `RAILS_MASTER_KEY`** — Without it, credentials can't be decrypted and the app won't boot.

## Best Practices
1. **Use the Solid Trifecta** — Solid Cache, Solid Queue, and Solid Cable eliminate the Redis dependency for most apps, simplifying infrastructure.
2. **Set database pool to match threads** — `pool` should equal `RAILS_MAX_THREADS` to prevent connection timeouts under load.

## Summary
- Rails 8 uses `config.enable_reloading = false` instead of the deprecated `cache_classes`.
- Solid Cache is the default cache store — database-backed, no Redis needed.
- Solid Queue is the default job backend — database-backed, no Redis needed.
- Store secrets in encrypted credentials, decrypted by `RAILS_MASTER_KEY`.
- Set database pool size to match your thread count.

## Code Examples

**Rails 8 production configuration — uses the Solid Trifecta (database-backed cache, queue, and cable) instead of Redis**

```ruby
# Rails 8 production defaults — no Redis needed!
Rails.application.configure do
  config.enable_reloading = false        # Don't reload code
  config.eager_load = true               # Load everything at boot
  config.force_ssl = true                # HTTPS only
  config.cache_store = :solid_cache_store # Database-backed cache
  config.active_job.queue_adapter = :solid_queue # Database-backed jobs
end
```


## Resources

- [Configuring Rails Applications](https://guides.rubyonrails.org/configuring.html) — Official Rails guide on production environment configuration

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*