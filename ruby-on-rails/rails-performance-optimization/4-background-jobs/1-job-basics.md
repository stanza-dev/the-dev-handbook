---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-active-job-basics"
---

# Active Job Basics

## Introduction
Background jobs improve response times by deferring slow work — sending emails, processing uploads, calling APIs — to a separate process. In Rails 8, Active Job is backed by Solid Queue by default, a database-backed queue that requires no additional infrastructure.

## Key Concepts
- **Active Job**: Rails' built-in framework for declaring and running background jobs with a unified API across queue backends.
- **Solid Queue**: Rails 8's default queue backend — stores jobs in the database, supports priorities, concurrency controls, and delayed execution.
- **perform_later vs perform_now**: `perform_later` enqueues the job for background processing; `perform_now` runs it immediately (blocking).

## Real World Context
A user registration that sends a welcome email, creates a Stripe customer, and syncs to a CRM takes 3 seconds synchronously. Moving those to background jobs reduces the response to 50ms — the user sees an instant redirect while jobs process behind the scenes.

## Deep Dive

### Creating a Job

```bash
bin/rails generate job ProcessImage
```

```ruby
class ProcessImageJob < ApplicationJob
  queue_as :default

  def perform(image_id)
    image = Image.find(image_id)
    image.create_thumbnails
    image.extract_metadata
    image.update!(processed: true)
  end
end
```

### Enqueuing Jobs

```ruby
# Process in background
ProcessImageJob.perform_later(image.id)

# Delay execution
ProcessImageJob.set(wait: 5.minutes).perform_later(image.id)
ProcessImageJob.set(wait_until: Date.tomorrow.noon).perform_later(image.id)
```

### Configuring Solid Queue (Rails 8 Default)

```ruby
# config/environments/production.rb
config.active_job.queue_adapter = :solid_queue
config.solid_queue.connects_to = { database: { writing: :queue } }
```

Solid Queue can run embedded in Puma (no separate process needed):

```bash
# Set in environment or config/deploy.yml
SOLID_QUEUE_IN_PUMA=true
```

### Passing Arguments

Jobs serialize arguments, so pass simple types:

```ruby
# GOOD: Pass IDs
OrderEmailJob.perform_later(order.id)

# AVOID: Passing full ActiveRecord objects
# OrderEmailJob.perform_later(order)  # Works but fragile
```

## Common Pitfalls
1. **Passing ActiveRecord objects instead of IDs** — Objects can change between enqueue and execution. Always pass IDs and re-fetch the record in the job.
2. **Not handling RecordNotFound** — The record might be deleted between enqueue and execution. Use `discard_on ActiveRecord::RecordNotFound` or handle it gracefully.

## Best Practices
1. **Use Solid Queue for most apps** — It's the Rails 8 default, requires no Redis, and handles most workloads. Only reach for Sidekiq if you need sub-second latency at massive scale.
2. **Move any operation over 100ms to a background job** — Email delivery, file processing, API calls, and report generation should all be async.

## Summary
- Active Job provides a unified API for background processing across queue backends.
- Rails 8 defaults to Solid Queue — database-backed, no Redis needed.
- Use `perform_later` to enqueue jobs asynchronously.
- Pass record IDs, not full objects, as job arguments.
- Solid Queue can run embedded in Puma with `SOLID_QUEUE_IN_PUMA=true`.

## Code Examples

**A simple background job that sends a welcome email — the controller responds instantly while the email sends in the background**

```ruby
class WelcomeEmailJob < ApplicationJob
  queue_as :default

  def perform(user_id)
    user = User.find(user_id)
    UserMailer.welcome(user).deliver_now
  end
end

# Enqueue in controller
WelcomeEmailJob.perform_later(@user.id)
```


## Resources

- [Active Job Basics](https://guides.rubyonrails.org/active_job_basics.html) — Official Rails guide on Active Job including Solid Queue configuration

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*