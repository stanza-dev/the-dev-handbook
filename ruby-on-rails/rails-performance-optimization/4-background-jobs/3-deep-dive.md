---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-solid-queue-deep-dive"
---

# Solid Queue Configuration

## Introduction
Solid Queue is Rails 8's default background job backend. It stores jobs in the database, supports concurrency controls, priority queues, and can run embedded in Puma — eliminating the need for a separate process or Redis in most applications.

## Key Concepts
- **Database-Backed Queue**: Jobs are persisted in database tables, surviving process restarts and deployments.
- **Embedded Mode**: Solid Queue can run inside the Puma web server process with `SOLID_QUEUE_IN_PUMA=true`, simplifying deployment.
- **Concurrency Controls**: Limit how many instances of a job type can run simultaneously to protect external APIs or shared resources.

## Real World Context
A typical Rails 8 app deploys with zero Redis dependency — Solid Queue handles jobs, Solid Cache handles caching, and Solid Cable handles WebSockets, all backed by the database. This reduces infrastructure cost and complexity.

## Deep Dive

### Configuration File

```yaml
# config/solid_queue.yml
default: &default
  dispatchers:
    - polling_interval: 1
      batch_size: 500
  workers:
    - queues: "*"
      threads: 5
      processes: 1
      polling_interval: 0.1

production:
  <<: *default
  workers:
    - queues: [critical, default]
      threads: 5
      processes: 2
    - queues: [low_priority]
      threads: 2
      processes: 1
```

### Database Setup

```yaml
# config/database.yml
production:
  primary:
    url: <%= ENV['DATABASE_URL'] %>
  queue:
    url: <%= ENV['QUEUE_DATABASE_URL'] %>
    migrations_paths: db/queue_migrate
```

```ruby
# config/environments/production.rb
config.active_job.queue_adapter = :solid_queue
config.solid_queue.connects_to = { database: { writing: :queue } }
```

### Embedded Mode (Run in Puma)

```yaml
# config/deploy.yml (Kamal)
env:
  clear:
    SOLID_QUEUE_IN_PUMA: true
```

This starts Solid Queue automatically when Puma boots — no separate worker process needed.

### Concurrency Controls

```ruby
class SyncToExternalApiJob < ApplicationJob
  limits_concurrency to: 2, key: ->(*args) { "api_sync" }

  def perform(record_id)
    ExternalApi.sync(Record.find(record_id))
  end
end
```

### Priority Queues

```ruby
class CriticalAlertJob < ApplicationJob
  queue_as :critical
end

class ReportGenerationJob < ApplicationJob
  queue_as :low_priority
end
```

## Common Pitfalls
1. **Not setting up a separate queue database** — For high-volume apps, job tables should use a separate database to avoid lock contention with your application data.
2. **Using embedded mode for heavy workloads** — Embedded mode is convenient for simple apps, but CPU-intensive jobs can starve web request threads.

## Best Practices
1. **Start with embedded mode** — For most apps, `SOLID_QUEUE_IN_PUMA=true` is the simplest deployment. Scale to separate workers when needed.
2. **Use concurrency controls for external APIs** — Prevent rate limiting by capping concurrent jobs that call the same external service.

## Summary
- Solid Queue stores jobs in the database — no Redis required.
- Embedded mode (`SOLID_QUEUE_IN_PUMA=true`) runs jobs inside Puma.
- Use a separate queue database for high-volume production apps.
- Concurrency controls prevent overwhelming external APIs.
- Configure priority queues to process critical jobs first.

## Code Examples

**Solid Queue concurrency control — limits concurrent API calls to prevent rate limiting**

```ruby
# Limit concurrent API calls to prevent rate limiting
class ExternalSyncJob < ApplicationJob
  limits_concurrency to: 3, key: ->(*args) { "external_api" }
  queue_as :default

  def perform(user_id)
    user = User.find(user_id)
    ExternalCrm.sync_user(user)
  end
end

# Only 3 instances of this job run simultaneously
```


## Resources

- [Solid Queue GitHub Repository](https://github.com/rails/solid_queue) — Official Solid Queue repository with configuration documentation and examples

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*