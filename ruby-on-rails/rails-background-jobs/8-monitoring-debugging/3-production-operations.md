---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-production-operations"
---

# Production Operations

## Introduction
Running background jobs in production requires operational practices beyond code: deploying without job loss, scaling workers, and handling backlogs.

## Key Concepts
- **Graceful Shutdown**: Workers finish current jobs before stopping.
- **Backlog Management**: Strategies for when jobs pile up faster than workers process them.
- **Deploy Safety**: Ensuring job compatibility during deployments.

## Real World Context
During a deploy, old workers stop and new ones start. If a worker is mid-job when stopped, what happens? Graceful shutdown lets it finish.

## Deep Dive

### Graceful Shutdown

Solid Queue and Sidekiq both support graceful shutdown. When receiving SIGTERM:

```ruby
# Solid Queue: workers finish current jobs, then exit
# Configure shutdown timeout:
# bin/jobs start --timeout 30

# Sidekiq: same behavior
# SIDEKIQ_TIMEOUT=25 bundle exec sidekiq
```

### Scaling Workers

```bash
# Scale Solid Queue workers via environment variable
JOB_CONCURRENCY=4 bin/jobs start

# Or adjust config/queue.yml:
# workers:
#   - queues: "*"
#     threads: 5
#     processes: 4
```

### Handling Backlogs

```ruby
# Check if a backlog is building
ready_count = SolidQueue::ReadyExecution.count
if ready_count > 10_000
  Rails.logger.warn "Job backlog: #{ready_count} jobs waiting"
  # Consider: add workers, pause non-critical enqueuing, or prioritize
end
```

### Deploy-Safe Jobs

```ruby
# If you rename a job class, old jobs in the queue will fail.
# Keep the old class as an alias during transition:
class OldJobName < ApplicationJob
  def perform(*args)
    NewJobName.perform_now(*args)
  end
end
```

## Common Pitfalls
1. **Deploying breaking changes to job arguments** — Old jobs in the queue have old arguments.
2. **Killing workers with SIGKILL instead of SIGTERM** — SIGKILL skips graceful shutdown.

## Best Practices
1. **Always use SIGTERM for worker shutdown** — Gives workers time to finish.
2. **Make job changes backward-compatible** — Old and new versions may coexist during deploys.

## Summary
- Use SIGTERM for graceful worker shutdown.
- Scale workers via processes and threads configuration.
- Monitor backlogs and scale workers proactively.
- Make job changes backward-compatible during deploys.

## Code Examples

**Checking job backlog size and oldest job latency**

```ruby
# Monitor backlog and alert
ready = SolidQueue::ReadyExecution.count
oldest = SolidQueue::ReadyExecution.order(:created_at).first
latency = oldest ? (Time.current - oldest.created_at).round : 0

puts "Backlog: #{ready} jobs, oldest waiting #{latency}s"
```


## Resources

- [Solid Queue](https://github.com/rails/solid_queue) — Solid Queue documentation

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*