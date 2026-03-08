---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-debugging-jobs"
---

# Debugging Jobs

## Introduction
When jobs fail, the debugging experience differs from web requests — no browser, no request log, failure may have happened hours ago.

## Key Concepts
- **Failed Execution**: Solid Queue record with error message and backtrace.
- **Structured Logging**: Including job metadata in every log entry.

## Real World Context
Order #4521 was never fulfilled. Find the failed job, understand why, fix and retry — or resolve manually.

## Deep Dive

### Structured Logging

```ruby
class ApplicationJob < ActiveJob::Base
  around_perform do |job, block|
    Rails.logger.tagged("job:#{job.class.name}", "jid:#{job.job_id}") do
      Rails.logger.info "Starting: #{job.arguments.inspect}"
      block.call
      Rails.logger.info "Completed"
    end
  rescue => e
    Rails.logger.error "Failed: #{e.message}"
    raise
  end
end
```

### Inspecting Failed Jobs (Solid Queue)

```ruby
SolidQueue::FailedExecution.last(10).each do |fe|
  puts "#{fe.job.class_name}: #{fe.error['message']}"
  puts "Args: #{fe.job.arguments}"
end

# Retry a failed job
SolidQueue::FailedExecution.find(123).retry

# Retry all
SolidQueue::FailedExecution.find_each(&:retry)
```

### Inspecting Failed Jobs (Sidekiq)

```ruby
rs = Sidekiq::RetrySet.new
rs.each { |job| puts "#{job.klass}: #{job.error_message}" }

ds = Sidekiq::DeadSet.new
ds.each { |job| puts "#{job.klass}: #{job.error_message}" }

# Retry specific job
rs.find { |j| j.jid == 'abc123' }.retry
```

## Common Pitfalls
1. **Not logging job arguments** — Can't reproduce without knowing the inputs.
2. **Deleting failed jobs without investigation** — They contain valuable debugging info.

## Best Practices
1. **Add structured logging to ApplicationJob.**
2. **Keep failed jobs for at least 7 days.**

## Summary
- Add structured logging for consistent, filterable output.
- Use SolidQueue::FailedExecution or Sidekiq::RetrySet to inspect failures.
- Retry individual or all failed jobs from the console.
- Keep failed jobs for investigation before discarding.

## Code Examples

**Inspecting and retrying failed jobs in Solid Queue**

```ruby
# Inspect and retry failed Solid Queue jobs
SolidQueue::FailedExecution.last(5).each do |fe|
  puts "#{fe.job.class_name}: #{fe.error['message']}"
end
SolidQueue::FailedExecution.find(42).retry
```


## Resources

- [Solid Queue Failed Jobs](https://github.com/rails/solid_queue#failed-jobs-and-retries) — Handling failed jobs

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*