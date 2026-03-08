---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-creating-jobs"
---

# Creating and Enqueuing Jobs

## Introduction
Rails provides generators and a clean API for creating jobs and controlling when they execute. Understanding the enqueuing options lets you schedule work precisely.

## Key Concepts
- **perform_later**: Enqueues the job for asynchronous execution by a worker process.
- **perform_now**: Executes the job immediately and synchronously in the current process.
- **set**: A chainable method that configures options like wait time or queue before enqueuing.
- **Global ID**: Rails' built-in serialization for Active Record objects.

## Real World Context
In a real application, you enqueue jobs from controllers after user actions, from model callbacks after data changes, and from other jobs to build workflows. Knowing the difference between `perform_later`, `perform_now`, and `set(wait:)` is essential.

## Deep Dive

### Generating a Job

```bash
bin/rails generate job ProcessImage
```

This creates `app/jobs/process_image_job.rb`:

```ruby
class ProcessImageJob < ApplicationJob
  queue_as :default

  def perform(*args)
    # Job logic here
  end
end
```

### Enqueuing Options

```ruby
# Execute as soon as a worker is available
ProcessImageJob.perform_later(image.id)

# Execute after a delay
ProcessImageJob.set(wait: 5.minutes).perform_later(image.id)

# Execute at a specific time
ReportJob.set(wait_until: Date.tomorrow.noon).perform_later

# Execute immediately (synchronous)
ProcessImageJob.perform_now(image.id)
```

### Passing Arguments

```ruby
# GOOD: Pass IDs and primitives
SendReportJob.perform_later(user.id, "weekly", Date.today.to_s)

# GOOD: Global ID serialization
SendReportJob.perform_later(user)  # Serialized as gid://app/User/42

# AVOID: Complex objects
SendReportJob.perform_later(order.attributes)  # Stale data risk
```

### Queue Assignment

```ruby
class PaymentJob < ApplicationJob
  queue_as :critical
end

class ReportJob < ApplicationJob
  queue_as :low_priority
end
```

### Job Callbacks

```ruby
class AuditedJob < ApplicationJob
  before_enqueue :log_enqueue
  before_perform :log_start
  after_perform :log_complete

  def perform(record_id)
    # Job logic
  end

  private

  def log_enqueue
    Rails.logger.info "Enqueueing #{self.class.name}"
  end
end
```

## Common Pitfalls
1. **Using perform_now in production controllers** — This blocks the web request. Reserve it for console, tests, and rake tasks.
2. **Passing data that changes between enqueue and execution** — Pass the record ID and re-fetch inside the job.

## Best Practices
1. **Check if records still exist** — A user might be deleted between enqueue and execution.
2. **Use meaningful queue names** — Names like `:critical`, `:mailers`, `:imports` make worker allocation clear.

## Summary
- Use `bin/rails generate job` to create job classes.
- `perform_later` enqueues asynchronously; `perform_now` executes synchronously.
- `set(wait:)` and `set(wait_until:)` schedule delayed execution.
- Pass IDs or serializable primitives as job arguments.
- Use `queue_as` to assign jobs to priority-based queues.

## Code Examples

**Scheduling jobs with delays — set(wait:) for relative delays, set(wait_until:) for absolute times**

```ruby
# Schedule a reminder 24 hours from now
ReminderJob.set(wait: 24.hours).perform_later(user.id, "Complete your profile")

# Schedule for next Monday at 9 AM
ReportJob.set(wait_until: Date.today.next_occurring(:monday).in_time_zone.change(hour: 9)).perform_later
```


## Resources

- [Active Job — Enqueue the Job](https://guides.rubyonrails.org/active_job_basics.html#enqueue-the-job) — Official guide on enqueuing and executing jobs

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*