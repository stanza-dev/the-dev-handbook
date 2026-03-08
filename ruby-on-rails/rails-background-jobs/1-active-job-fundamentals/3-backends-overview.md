---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-job-backends-overview"
---

# Job Backend Options in Rails 8

## Introduction
Active Job is a framework-level abstraction — it defines how you write and enqueue jobs, but delegates actual queuing to a backend adapter. Rails 8 ships with Solid Queue as the default, but several alternatives exist.

## Key Concepts
- **Queue Adapter**: The backend that stores and processes jobs. Configured via `config.active_job.queue_adapter`.
- **Solid Queue**: The default database-backed queue in Rails 8+. No external dependencies.
- **Sidekiq**: A popular Redis-backed queue known for high throughput.
- **Async Adapter**: In-process, keeps jobs in RAM. Development only.

## Real World Context
Choosing a queue backend affects deployment complexity, reliability, and operational overhead. With Rails 8, Solid Queue works out of the box. You only need Sidekiq when you have specific throughput requirements.

## Deep Dive

### Solid Queue (Rails 8 Default)

Solid Queue stores jobs in your existing database:

```ruby
# config/environments/production.rb
config.active_job.queue_adapter = :solid_queue
config.solid_queue.connects_to = { database: { writing: :queue } }
```

Key features: no external dependencies, concurrency controls, recurring jobs, numeric priorities.

### Sidekiq (Redis-backed)

```ruby
# Gemfile
gem "sidekiq"

# config/application.rb
config.active_job.queue_adapter = :sidekiq
```

### Other Backends

| Backend | Storage | Best For |
|---------|---------|----------|
| Solid Queue | Database | Most Rails apps (default) |
| Sidekiq | Redis | High throughput |
| Good Job | PostgreSQL | PG-only stacks |
| Delayed Job | Database | Simple apps |

### Development Adapter

```ruby
# config/environments/development.rb
config.active_job.queue_adapter = :async  # In-process, jobs lost on restart
```

## Common Pitfalls
1. **Running async adapter in production** — It loses all jobs on restart and cannot scale across servers.
2. **Choosing Sidekiq when you don't need Redis** — Solid Queue handles most workloads without extra infrastructure.

## Best Practices
1. **Start with Solid Queue** — It is the Rails default, requires no additional infrastructure.
2. **Match your backend to your infrastructure** — Already using Redis? Sidekiq is low overhead.

## Summary
- Active Job abstracts the queue backend so job classes work with any adapter.
- Solid Queue is the Rails 8 default — database-backed, no external dependencies.
- Sidekiq is the most popular Redis-backed alternative.
- The async adapter is for development only.

## Code Examples

**Configuring Solid Queue as the production job backend — the Rails 8 default requires no external services**

```ruby
# config/environments/production.rb
config.active_job.queue_adapter = :solid_queue
config.solid_queue.connects_to = { database: { writing: :queue } }

# Install Solid Queue
# bin/rails solid_queue:install
```


## Resources

- [Active Job Basics — Backends](https://guides.rubyonrails.org/active_job_basics.html#queue-backend-support) — Official guide covering queue adapter selection
- [Solid Queue Repository](https://github.com/rails/solid_queue) — Solid Queue source code and documentation

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*