---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-installing-solid-queue"
---

# Installing Solid Queue

## Introduction
Solid Queue is the default job backend in Rails 8, storing jobs in your database via Active Record. Setup takes a few commands with no external services required.

## Key Concepts
- **solid_queue:install**: A Rails generator that sets up the schema, configuration, and Puma plugin.
- **queue.yml**: Configuration file defining dispatchers and workers.
- **Database-backed**: Jobs stored as rows, durable and queryable.

## Real World Context
New Rails 8 apps have Solid Queue configured automatically. For existing apps upgrading, a single install command sets everything up.

## Deep Dive

### Installation

```bash
bin/rails solid_queue:install
```

This creates:
- `db/queue_schema.rb` — database schema for job tables
- `config/queue.yml` — worker and dispatcher configuration
- Updates `config/database.yml` to add a queue database

### Database Configuration

```yaml
# config/database.yml
production:
  primary:
    <<: *default
    database: myapp_production
  queue:
    <<: *default
    database: myapp_production_queue
    migrations_paths: db/queue_migrate
```

```ruby
# config/environments/production.rb
config.active_job.queue_adapter = :solid_queue
config.solid_queue.connects_to = { database: { writing: :queue } }
```

### Running Solid Queue

```bash
# Option 1: Puma plugin (automatic)
plugin :solid_queue  # in config/puma.rb

# Option 2: Standalone process
bin/jobs start
```

## Common Pitfalls
1. **Forgetting to create the queue database** — Run `bin/rails db:prepare` after installation.
2. **Using SQLite in production** — Use PostgreSQL or MySQL for concurrent access.

## Best Practices
1. **Use a separate database for the queue** — Prevents job locks from affecting application queries.
2. **Run workers as separate processes in production** — Scale independently of web servers.

## Summary
- Install with `bin/rails solid_queue:install`.
- Jobs stored in a dedicated queue database.
- Puma plugin starts workers automatically, or run `bin/jobs start`.
- Always use PostgreSQL or MySQL in production.

## Code Examples

**Production configuration for Solid Queue**

```ruby
# config/environments/production.rb
config.active_job.queue_adapter = :solid_queue
config.solid_queue.connects_to = { database: { writing: :queue } }
```


## Resources

- [Active Job — Solid Queue](https://guides.rubyonrails.org/active_job_basics.html#default-backend-solid-queue) — Official Rails guide on Solid Queue
- [Solid Queue Repository](https://github.com/rails/solid_queue) — Solid Queue source code

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*