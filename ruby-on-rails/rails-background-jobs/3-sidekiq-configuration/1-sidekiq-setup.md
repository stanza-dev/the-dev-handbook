---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-sidekiq-setup"
---

# Setting Up Sidekiq

## Introduction
Sidekiq is the most popular Redis-backed job backend for Rails. If your application already uses Redis or requires Sidekiq-specific features like a real-time web dashboard and high throughput, it is a proven alternative to Solid Queue.

## Key Concepts
- **Sidekiq**: A multi-threaded, Redis-backed background job processor.
- **Redis**: An in-memory data store used by Sidekiq for job queuing.
- **Sidekiq::Web**: A built-in web dashboard for monitoring queues and jobs.

## Real World Context
Sidekiq is used by companies processing millions of jobs per day. It provides sub-millisecond enqueue times and a rich plugin ecosystem.

## Deep Dive

### Installation

```ruby
# Gemfile
gem 'sidekiq'
```

```ruby
# config/application.rb
config.active_job.queue_adapter = :sidekiq
```

### Configuration

```yaml
# config/sidekiq.yml
:concurrency: 10
:queues:
  - [critical, 3]
  - [default, 2]
  - [low, 1]

production:
  :concurrency: 25
```

### Redis Configuration

```ruby
# config/initializers/sidekiq.rb
Sidekiq.configure_server do |config|
  config.redis = { url: ENV.fetch('REDIS_URL', 'redis://localhost:6379/0') }
end

Sidekiq.configure_client do |config|
  config.redis = { url: ENV.fetch('REDIS_URL', 'redis://localhost:6379/0') }
end
```

### Running Sidekiq

```bash
bundle exec sidekiq
bundle exec sidekiq -C config/sidekiq.yml
```

### Web UI

```ruby
# config/routes.rb
require 'sidekiq/web'

Rails.application.routes.draw do
  authenticate :user, ->(u) { u.admin? } do
    mount Sidekiq::Web => '/sidekiq'
  end
end
```

## Common Pitfalls
1. **Not protecting the Sidekiq web dashboard** — Without authentication, anyone can view job data and retry jobs.
2. **Ignoring Redis memory** — Redis stores everything in memory. Monitor usage closely.

## Best Practices
1. **Use a dedicated Redis instance** — Separate from cache Redis to prevent data loss during evictions.
2. **Set queue weights thoughtfully** — `[critical, 3]` means critical is polled 3x more often.

## Summary
- Sidekiq is Redis-backed with a mature ecosystem.
- Configure via `config/sidekiq.yml` for concurrency and queues.
- Always protect the web dashboard with authentication.
- Use a dedicated Redis instance in production.

## Code Examples

**Sidekiq Redis configuration for server and client**

```ruby
# config/initializers/sidekiq.rb
Sidekiq.configure_server do |config|
  config.redis = { url: ENV.fetch('REDIS_URL', 'redis://localhost:6379/0') }
end

Sidekiq.configure_client do |config|
  config.redis = { url: ENV.fetch('REDIS_URL', 'redis://localhost:6379/0') }
end
```


## Resources

- [Sidekiq Wiki](https://github.com/sidekiq/sidekiq/wiki) — Comprehensive Sidekiq documentation

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*