---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-scheduled-jobs"
---

# Scheduling Jobs

## Introduction
Not every job should run immediately. Active Job provides `set(wait:)` and `set(wait_until:)` to schedule jobs for future execution.

## Key Concepts
- **set(wait:)**: Schedule after a relative duration.
- **set(wait_until:)**: Schedule at a specific time.
- **Job Chains**: Jobs that schedule follow-up jobs.

## Real World Context
After signup, send welcome email immediately but schedule onboarding tips for 3 days later. After an order ships, schedule a review request for 7 days out.

## Deep Dive

### Delayed Execution

```ruby
ReminderJob.set(wait: 24.hours).perform_later(user.id)
ReportJob.set(wait_until: Date.tomorrow.noon).perform_later
```

### Job Chaining

```ruby
class OnboardingSequenceJob < ApplicationJob
  def perform(user_id, step)
    user = User.find_by(id: user_id)
    return unless user

    case step
    when 1
      OnboardingMailer.tips(user).deliver_now
      self.class.set(wait: 3.days).perform_later(user_id, 2)
    when 2
      OnboardingMailer.features(user).deliver_now
      self.class.set(wait: 7.days).perform_later(user_id, 3)
    when 3
      OnboardingMailer.feedback_request(user).deliver_now
    end
  end
end
```

### Processing Chain

```ruby
class ProcessOrderJob < ApplicationJob
  def perform(order_id)
    order = Order.find(order_id)
    order.process!
    SendConfirmationJob.perform_later(order.id)
    SyncToWarehouseJob.set(wait: 5.minutes).perform_later(order.id)
  end
end
```

## Common Pitfalls
1. **Scheduling far-future jobs without guards** — Records may be deleted. Always check existence.
2. **Using wall-clock time without time zones** — Use `in_time_zone` for user-specific scheduling.

## Best Practices
1. **Guard against stale references** — Check records still exist in delayed jobs.
2. **Log scheduled times** — Helps debug timing issues.

## Summary
- `set(wait:)` for relative delays, `set(wait_until:)` for absolute times.
- Jobs can schedule follow-up jobs to create sequences.
- Always check record existence in delayed jobs.

## Code Examples

**A drip email job scheduling the next step with a 3-day delay**

```ruby
# Drip email sequence
class DrippEmailJob < ApplicationJob
  def perform(user_id, step)
    user = User.find_by(id: user_id)
    return unless user&.subscribed?

    DrippMailer.send("email_#{step}", user).deliver_now
    self.class.set(wait: 3.days).perform_later(user_id, step + 1) if step < 5
  end
end
```


## Resources

- [Active Job — Enqueue](https://guides.rubyonrails.org/active_job_basics.html#enqueue-the-job) — Official guide on enqueuing with delays

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*