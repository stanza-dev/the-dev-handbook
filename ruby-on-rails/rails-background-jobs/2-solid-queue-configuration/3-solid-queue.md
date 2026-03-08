---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-recurring-jobs-solid-queue"
---

# Recurring Jobs with Solid Queue

## Introduction
Solid Queue includes built-in support for recurring jobs — tasks that run on a schedule without needing additional gems.

## Key Concepts
- **Recurring Task**: A job executing on a cron-like schedule.
- **recurring.yml**: Configuration file for scheduled job entries.
- **Cron Expression**: Standard notation for execution schedules.

## Real World Context
Every app needs recurring work: daily reports, hourly cleanup, weekly digests. With Solid Queue, these schedules live alongside your job configuration.

## Deep Dive

### Defining Recurring Jobs

```yaml
# config/recurring.yml
production:
  cleanup_sessions:
    class: CleanupSessionsJob
    queue: low_priority
    schedule: "0 3 * * *"  # Daily at 3 AM

  daily_digest:
    class: DailyDigestJob
    queue: mailers
    schedule: "0 8 * * 1-5"  # Weekdays at 8 AM

  hourly_sync:
    class: SyncInventoryJob
    queue: default
    schedule: "0 * * * *"  # Every hour
```

### Job Classes

Recurring job classes are standard Active Job classes:

```ruby
class DailyDigestJob < ApplicationJob
  queue_as :mailers

  def perform
    User.subscribed_to_digest.find_each do |user|
      DigestMailer.daily(user).deliver_later
    end
  end
end
```

## Common Pitfalls
1. **Not preventing overlap** — Use `limits_concurrency` if jobs take longer than their interval.
2. **Forgetting environment-specific schedules** — Jobs under `production:` won't run in development.

## Best Practices
1. **Make recurring jobs fast or self-limiting** — Use `find_each` for batching.
2. **Log completion** — Recurring jobs have no caller, so logging is your visibility.

## Summary
- Solid Queue supports recurring jobs via `config/recurring.yml`.
- Define schedules using cron expressions.
- Use `limits_concurrency` to prevent overlapping executions.

## Code Examples

**Recurring jobs in Solid Queue with cron expressions**

```yaml
# config/recurring.yml
production:
  cleanup_tokens:
    class: CleanupTokensJob
    queue: low_priority
    schedule: "0 4 * * *"
  weekly_digest:
    class: WeeklyDigestJob
    queue: mailers
    schedule: "0 9 * * 1"
```


## Resources

- [Solid Queue Recurring Tasks](https://github.com/rails/solid_queue#recurring-tasks) — Recurring tasks documentation

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*