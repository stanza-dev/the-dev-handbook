---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-active-job-basics"
---

# Active Job Basics

Background jobs improve response times by deferring slow work.

## When to Use Background Jobs

- Sending emails
- Processing uploads
- Calling external APIs
- Generating reports
- Data imports/exports
- Any operation taking > 100ms

## Creating a Job

```bash
bin/rails generate job ProcessImage
```

```ruby
# app/jobs/process_image_job.rb
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

## Enqueuing Jobs

```ruby
# Process later (in background queue)
ProcessImageJob.perform_later(image.id)

# Process at specific time
ProcessImageJob.set(wait: 5.minutes).perform_later(image.id)
ProcessImageJob.set(wait_until: Date.tomorrow.noon).perform_later(image.id)

# Process immediately (blocking - for testing)
ProcessImageJob.perform_now(image.id)
```

## Queue Configuration

```ruby
class ProcessImageJob < ApplicationJob
  queue_as :images
  
  # Or dynamically
  queue_as do
    if self.arguments.first.priority == 'high'
      :urgent
    else
      :default
    end
  end
end
```

## Configuring Backend

```ruby
# config/application.rb
config.active_job.queue_adapter = :sidekiq

# Other options: :async, :inline, :delayed_job, :resque
```

## Passing Arguments

Jobs serialize arguments, so pass simple types:

```ruby
# GOOD: Pass IDs
OrderEmailJob.perform_later(order.id)

# AVOID: Passing ActiveRecord objects
# OrderEmailJob.perform_later(order)  # Works but fragile
```

See [Active Job Guide](https://guides.rubyonrails.org/active_job_basics.html).

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*