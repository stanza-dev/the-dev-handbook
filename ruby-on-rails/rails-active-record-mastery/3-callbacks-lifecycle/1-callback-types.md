---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-callback-types"
---

# Understanding Callbacks

## Introduction
Callbacks are methods that get called at specific points in an object's lifecycle. They let you trigger logic during creation, updates, saves, and deletion. Understanding the callback chain is essential for writing predictable Active Record models.

## Key Concepts
- **Callback**: A method that is automatically invoked at a specific point in the lifecycle of an Active Record object.
- **Callback chain**: The ordered sequence of callbacks that runs during operations like `save`, `create`, `update`, and `destroy`.
- **`throw(:abort)`**: The mechanism to halt the callback chain and cancel the operation.
- **`around_*` callbacks**: Wrap the operation, giving you control before and after the database action within a single method.

## Real World Context
Callbacks automate side effects that should always happen when data changes: normalizing emails before validation, setting default values before save, sending notifications after creation, or cleaning up files after deletion. Every production Rails app uses callbacks, and misusing them is a common source of bugs.

## Deep Dive

### The Callback Chain

When you save a new record, Rails runs callbacks in this precise order:

```
before_validation
after_validation
before_save
around_save
before_create
around_create
[INSERT INTO database]
after_create
after_save
after_commit / after_rollback
```

For updates, `before_create`/`after_create` are replaced by `before_update`/`after_update`. For destruction, the chain is `before_destroy`, `around_destroy`, DELETE, `after_destroy`, then `after_commit`.

### Defining Callbacks

```ruby
class Article < ApplicationRecord
  # Method reference
  before_save :set_published_at
  after_create :notify_subscribers

  # Block syntax
  before_validation { self.title = title.strip if title }

  private

  def set_published_at
    self.published_at ||= Time.current if published?
  end

  def notify_subscribers
    NotificationJob.perform_later(self)
  end
end
```

The method reference style (`:set_published_at`) is preferred over blocks for anything longer than one line.

### Common Use Cases

#### Before Validation -- Normalizing Data

```ruby
class User < ApplicationRecord
  before_validation :normalize_email

  private

  def normalize_email
    self.email = email.downcase.strip if email
  end
end
```

This runs before validation, so the normalized email is what gets validated.

#### Before Save -- Setting Defaults

```ruby
class Post < ApplicationRecord
  before_save :set_slug
  before_save :calculate_reading_time

  private

  def set_slug
    self.slug ||= title.parameterize
  end

  def calculate_reading_time
    self.reading_time = (body.split.size / 200.0).ceil
  end
end
```

#### Before Destroy -- Preventing Deletion

```ruby
class User < ApplicationRecord
  before_destroy :check_for_orders

  private

  def check_for_orders
    if orders.any?
      errors.add(:base, "Cannot delete user with orders")
      throw(:abort)  # Prevents deletion
    end
  end
end
```

### Halting the Callback Chain

```ruby
class Article < ApplicationRecord
  before_save :check_spam

  private

  def check_spam
    if SpamDetector.spam?(body)
      errors.add(:body, "contains spam")
      throw(:abort)  # Stops save and remaining callbacks
    end
  end
end
```

Calling `throw(:abort)` stops the entire operation. The record will not be saved, and no further callbacks will run.

## Common Pitfalls
1. **Too much logic in callbacks** -- Callbacks should be thin. If a callback grows beyond 5-10 lines, extract it into a service object.
2. **Side effects in before_save** -- Sending emails or making API calls in `before_save` is dangerous because the transaction may roll back. Use `after_commit` for side effects.
3. **Returning false instead of throw(:abort)** -- In modern Rails, returning `false` does NOT halt the chain. You must use `throw(:abort)`.

## Best Practices
1. **Keep callbacks focused** -- Each callback should do one thing. Use descriptive names like `normalize_email`, not `process_data`.
2. **Use `after_commit` for side effects** -- Emails, background jobs, and external API calls should use `after_commit` to ensure the transaction succeeded.
3. **Prefer explicit service calls over implicit callbacks** -- For complex multi-step operations, a service object is more testable and predictable than a chain of callbacks.

## Summary
- Callbacks run at specific points in the Active Record lifecycle (validation, save, create, update, destroy).
- Use `throw(:abort)` to halt the callback chain and cancel the operation.
- `before_save` runs for both creates and updates; `before_create`/`before_update` are specific.
- Keep callbacks thin and focused on a single responsibility.
- Use `after_commit` (not `after_save`) for side effects like emails and background jobs.

## Code Examples

**Common callback patterns -- normalizing data before validation, setting defaults before save, and triggering jobs after commit**

```ruby
class Article < ApplicationRecord
  before_validation { self.title = title.strip if title }
  before_save :set_published_at
  after_create_commit :notify_subscribers

  private

  def set_published_at
    self.published_at ||= Time.current if published?
  end

  def notify_subscribers
    NotificationJob.perform_later(self)
  end
end
```


## Resources

- [Active Record Callbacks](https://guides.rubyonrails.org/active_record_callbacks.html) — Complete guide to Active Record callbacks

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*