---
source_course: "rails-background-processing"
source_lesson: "rails-background-jobs-scheduled-jobs"
---

# Scheduled Jobs

Schedule jobs for delayed execution or recurring tasks.

## Delayed Execution

```ruby
# Execute in 5 minutes
ReminderJob.set(wait: 5.minutes).perform_later(user.id)

# Execute at specific time
ReportJob.set(wait_until: Date.tomorrow.noon).perform_later

# Execute on business days only
notification_time = 2.days.from_now
notification_time = notification_time.next_weekday if notification_time.on_weekend?
NotificationJob.set(wait_until: notification_time).perform_later(id)
```

## Sidekiq-Cron for Recurring Jobs

```ruby
# Gemfile
gem 'sidekiq-cron'
```

```yaml
# config/sidekiq_cron.yml
cleanup_job:
  cron: '0 3 * * *'  # Every day at 3 AM
  class: CleanupJob
  queue: low

daily_report:
  cron: '0 8 * * 1-5'  # Weekdays at 8 AM
  class: DailyReportJob
  args:
    - report_type: summary

hourly_sync:
  cron: '0 * * * *'  # Every hour
  class: SyncInventoryJob
```

```ruby
# config/initializers/sidekiq_cron.rb
if Sidekiq.server?
  schedule_file = 'config/sidekiq_cron.yml'
  
  if File.exist?(schedule_file)
    Sidekiq::Cron::Job.load_from_hash(YAML.load_file(schedule_file))
  end
end
```

## Job Chaining

```ruby
class ProcessOrderJob < ApplicationJob
  def perform(order_id)
    order = Order.find(order_id)
    order.process!
    
    # Chain next jobs
    SendConfirmationJob.perform_later(order.id)
    SyncToWarehouseJob.set(wait: 5.minutes).perform_later(order.id)
  end
end
```

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-processing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*