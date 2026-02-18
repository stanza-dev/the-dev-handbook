---
source_course: "rails-background-processing"
source_lesson: "rails-background-jobs-creating-jobs"
---

# Creating Jobs

Learn to create jobs and control when they execute.

## Generating Jobs

```bash
bin/rails generate job ProcessImage
```

```ruby
# app/jobs/process_image_job.rb
class ProcessImageJob < ApplicationJob
  queue_as :default
  
  def perform(image_id)
    image = Image.find(image_id)
    # Process the image
  end
end
```

## Enqueuing Jobs

```ruby
# Execute as soon as worker is available
ProcessImageJob.perform_later(image.id)

# Execute after delay
ProcessImageJob.set(wait: 5.minutes).perform_later(image.id)
ProcessImageJob.set(wait_until: Date.tomorrow.noon).perform_later(image.id)

# Execute immediately (blocking - for testing)
ProcessImageJob.perform_now(image.id)
```

## Passing Arguments

Jobs serialize arguments, so pass simple types:

```ruby
# GOOD: Pass IDs and primitives
SendReportJob.perform_later(user.id, Date.today.to_s)

# GOOD: Global ID (automatic serialization)
SendReportJob.perform_later(user)  # Serializes to gid://app/User/123

# AVOID: Complex objects that may change
SendReportJob.perform_later(order.attributes)  # Stale data risk
```

## Queue Assignment

```ruby
class ImportDataJob < ApplicationJob
  queue_as :imports  # Static queue
end

class PriorityJob < ApplicationJob
  queue_as do
    if self.arguments.first.priority == 'high'
      :urgent
    else
      :default
    end
  end
end
```

## Job Callbacks

```ruby
class ReportJob < ApplicationJob
  before_enqueue :log_enqueue
  before_perform :log_start
  after_perform :log_complete
  
  def perform(report_id)
    # Generate report
  end
  
  private
  
  def log_enqueue
    Rails.logger.info "Enqueueing report job"
  end
end
```

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-processing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*