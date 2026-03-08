---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-understanding-background-jobs"
---

# Understanding Background Jobs

## Introduction
Background jobs move time-consuming tasks out of the HTTP request-response cycle so your users get fast responses. Every production Rails application relies on background processing for emails, file handling, API integrations, and more.

## Key Concepts
- **Background Job**: A unit of work that executes outside the web request lifecycle, in a separate process or thread.
- **Job Queue**: A data structure (backed by a database or Redis) that holds jobs waiting to be processed.
- **Worker**: A process that pulls jobs from the queue and executes them.
- **Enqueuing**: The act of adding a job to the queue for later processing.

## Real World Context
Web requests should complete in under 200ms. Sending an email via SMTP can take 2-5 seconds. Resizing an uploaded image might take 10 seconds. Without background jobs, your users stare at a spinner while these operations block the request. Every Rails application — GitHub, Shopify, Basecamp — runs thousands of background jobs per minute.

## Deep Dive
The fundamental pattern is simple: instead of doing slow work during a web request, you enqueue a job that describes the work, return a fast response to the user, and let a separate worker process handle it.

```
User Request → Controller → Enqueue Job → Response (fast!)
                              ↓
                        Job Queue (DB/Redis)
                              ↓
                        Worker Process → Execute Job
```

Common use cases for background jobs:

- **Email delivery**: SMTP connections are slow and unreliable
- **File processing**: Image resizing, PDF generation, CSV imports
- **External API calls**: Third-party services may be slow or rate-limited
- **Data aggregation**: Report generation, analytics computation
- **Webhooks**: Outgoing HTTP requests to partner systems

Rails provides Active Job as a unified interface:

```ruby
class SendWelcomeEmailJob < ApplicationJob
  queue_as :default

  def perform(user_id)
    user = User.find(user_id)
    UserMailer.welcome(user).deliver_now
  end
end

# Enqueue the job — returns immediately
SendWelcomeEmailJob.perform_later(user.id)
```

## Common Pitfalls
1. **Passing Active Record objects instead of IDs** — Jobs serialize their arguments. Passing the ID is more explicit and avoids stale-data surprises.
2. **Doing too much in a single job** — Break complex workflows into smaller, focused jobs.
3. **Forgetting that jobs run in a different process** — Instance variables, request context, and current_user are not available.

## Best Practices
1. **Make jobs idempotent** — A job should produce the same result whether it runs once or five times.
2. **Keep job arguments simple and serializable** — Pass IDs, strings, numbers.
3. **Use appropriate queues** — Separate urgent jobs from slow jobs.

## Summary
- Background jobs move slow work out of web requests for faster user responses.
- Active Job provides a unified Ruby API across multiple queue backends.
- Jobs are enqueued with `perform_later` and executed by worker processes.
- Always pass simple, serializable arguments (prefer IDs over full objects).
- Design jobs to be idempotent so they can safely be retried.

## Code Examples

**A typical background job that processes an uploaded image — the controller returns immediately while the job runs asynchronously**

```ruby
class ProcessImageJob < ApplicationJob
  queue_as :default

  def perform(image_id)
    image = Image.find(image_id)
    image.variant(resize_to_limit: [800, 800]).processed
    image.update!(processed: true)
  end
end

# Enqueue from a controller
ProcessImageJob.perform_later(image.id)
```


## Resources

- [Active Job Basics](https://guides.rubyonrails.org/active_job_basics.html) — Official Rails guide covering Active Job fundamentals

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*