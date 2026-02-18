---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-production-configuration"
---

# Production Configuration

Production requires specific configuration for security, performance, and reliability.

## Environment Configuration

```ruby
# config/environments/production.rb
Rails.application.configure do
  # Code is not reloaded
  config.cache_classes = true
  config.eager_load = true
  
  # Full error reports disabled
  config.consider_all_requests_local = false
  
  # Caching
  config.action_controller.perform_caching = true
  config.cache_store = :redis_cache_store, {
    url: ENV['REDIS_URL'],
    expires_in: 1.day
  }
  
  # Force SSL
  config.force_ssl = true
  
  # Logging
  config.log_level = :info
  config.log_tags = [:request_id]
  
  # Use a real queuing backend
  config.active_job.queue_adapter = :sidekiq
  
  # Email configuration
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

## Credentials Management

```bash
# Edit production credentials
RAILS_MASTER_KEY=xxx bin/rails credentials:edit --environment production
```

```yaml
# config/credentials/production.yml.enc (decrypted)
secret_key_base: abc123...

database:
  password: db_password

redis:
  url: redis://redis:6379/1

aws:
  access_key_id: AKIA...
  secret_access_key: secret
```

## Database Configuration

```yaml
# config/database.yml
production:
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch('RAILS_MAX_THREADS', 5) %>
  url: <%= ENV['DATABASE_URL'] %>
  prepared_statements: true
```

## Environment Variables

Required variables for production:

```bash
RAILS_ENV=production
RAILS_MASTER_KEY=your_master_key
DATABASE_URL=postgres://user:pass@host:5432/db
REDIS_URL=redis://redis:6379/1
SECRET_KEY_BASE=generated_secret
RAILS_SERVE_STATIC_FILES=true
```

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*