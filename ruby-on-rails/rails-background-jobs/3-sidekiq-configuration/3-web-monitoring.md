---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-sidekiq-web-monitoring"
---

# Sidekiq Web Dashboard

## Introduction
Sidekiq includes a built-in web dashboard that provides real-time visibility into your job processing. It shows queue depths, processing rates, retry counts, and lets you manually retry or delete jobs.

## Key Concepts
- **Sidekiq::Web**: A Rack application you mount in your Rails routes.
- **Dashboard Metrics**: Processed, failed, busy, enqueued, retries, dead counts.
- **Manual Intervention**: Retry failed jobs, delete dead jobs, or clear queues from the UI.

## Real World Context
When a third-party API goes down and 500 jobs land in the retry queue, the Sidekiq dashboard lets you see the error messages, wait for the API to recover, and retry all failed jobs with one click.

## Deep Dive

### Mounting the Dashboard

```ruby
# config/routes.rb
require 'sidekiq/web'

Rails.application.routes.draw do
  authenticate :user, ->(u) { u.admin? } do
    mount Sidekiq::Web => '/sidekiq'
  end
end
```

### Dashboard Features

The dashboard shows:
- **Dashboard tab**: Real-time processed/failed graphs
- **Busy tab**: Currently executing jobs
- **Queues tab**: Queue sizes and ability to pause/delete
- **Retries tab**: Failed jobs scheduled for retry
- **Dead tab**: Jobs that exhausted all retries

### Programmatic Access

```ruby
# Same data available via API
stats = Sidekiq::Stats.new
stats.processed      # Total completed
stats.failed         # Total failures
stats.enqueued       # Waiting
stats.retry_size     # Pending retries
stats.dead_size      # Permanently failed
```

## Common Pitfalls
1. **Exposing the dashboard without authentication** — It shows job arguments which may contain sensitive data.
2. **Ignoring the Dead tab** — Dead jobs represent work that was never completed. Review regularly.

## Best Practices
1. **Set up alerts based on dashboard metrics** — Alert when retry_size or dead_size exceeds thresholds.
2. **Review dead jobs weekly** — Investigate patterns and fix root causes.

## Summary
- Mount Sidekiq::Web in routes behind authentication.
- The dashboard provides real-time metrics and manual intervention tools.
- Use Sidekiq::Stats for programmatic access to the same data.
- Always protect the dashboard with authentication.

## Code Examples

**A health check endpoint using Sidekiq stats**

```ruby
# Health check endpoint using Sidekiq stats
class HealthController < ApplicationController
  def sidekiq
    stats = Sidekiq::Stats.new
    healthy = stats.retry_size < 1000 && stats.dead_size < 100
    render json: { healthy: healthy, retry: stats.retry_size, dead: stats.dead_size },
           status: healthy ? :ok : :service_unavailable
  end
end
```


## Resources

- [Sidekiq Monitoring](https://github.com/sidekiq/sidekiq/wiki/Monitoring) — Sidekiq monitoring documentation

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*