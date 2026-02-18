---
source_course: "rails-background-processing"
source_lesson: "rails-background-jobs-understanding-background-jobs"
---

# Understanding Background Jobs

Background jobs move time-consuming tasks out of the request/response cycle.

## Why Background Jobs?

Web requests should be fast (< 200ms). Move slow operations to background:

- **Email sending**: SMTP is slow and unreliable
- **File processing**: Image resizing, PDF generation
- **External API calls**: Third-party services may be slow
- **Data imports/exports**: Large CSV processing
- **Report generation**: Complex calculations
- **Webhooks**: Outgoing HTTP requests

## How It Works

```
User Request → Controller → Enqueue Job → Response (fast!)
                              ↓
                        Job Queue (Redis)
                              ↓
                        Worker Process → Execute Job
```

## Active Job Overview

Active Job provides a unified interface across job backends:

```ruby
# Create a job
class SendEmailJob < ApplicationJob
  queue_as :default
  
  def perform(user_id)
    user = User.find(user_id)
    UserMailer.welcome(user).deliver_now
  end
end

# Enqueue it
SendEmailJob.perform_later(user.id)
```

## Job Backends

- **Sidekiq**: Redis-based, fast, popular (recommended)
- **Solid Queue**: Database-backed (Rails 8+)
- **Delayed Job**: Database-backed, simple
- **Resque**: Redis-based, fork per job
- **Good Job**: Postgres-based

## Configuration

```ruby
# config/application.rb
config.active_job.queue_adapter = :sidekiq

# Development: inline execution
# config/environments/development.rb
config.active_job.queue_adapter = :async
```

See [Active Job Basics](https://guides.rubyonrails.org/active_job_basics.html).

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-processing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*