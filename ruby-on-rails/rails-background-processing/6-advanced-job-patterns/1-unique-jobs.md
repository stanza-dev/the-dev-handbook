---
source_course: "rails-background-processing"
source_lesson: "rails-background-jobs-unique-jobs"
---

# Unique Jobs

Prevent the same job from running multiple times concurrently.

## Why Unique Jobs?

Without uniqueness, multiple identical jobs can run:

```ruby
# User clicks 'Send Email' multiple times
5.times { SendEmailJob.perform_later(user.id) }
# User receives 5 emails!
```

## DIY Unique Jobs with Redis

```ruby
class UniqueJob < ApplicationJob
  class DuplicateJobError < StandardError; end
  
  before_perform :acquire_lock
  after_perform :release_lock
  
  discard_on DuplicateJobError
  
  def unique_key
    [self.class.name, arguments].join(':')
  end
  
  def lock_timeout
    1.hour
  end
  
  private
  
  def acquire_lock
    acquired = Redis.current.set(
      "job_lock:#{unique_key}",
      job_id,
      nx: true,
      ex: lock_timeout.to_i
    )
    
    raise DuplicateJobError unless acquired
  end
  
  def release_lock
    # Only release if we hold the lock
    if Redis.current.get("job_lock:#{unique_key}") == job_id
      Redis.current.del("job_lock:#{unique_key}")
    end
  end
end

class ProcessOrderJob < UniqueJob
  def perform(order_id)
    # Only one instance runs per order_id
  end
  
  def unique_key
    "process_order:#{arguments.first}"
  end
end
```

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-processing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*