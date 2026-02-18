---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-transaction-callbacks"
---

# Transaction Callbacks

Transaction callbacks run after the database transaction is committed or rolled back. They're crucial for side effects that should only happen if the save actually succeeded.

## The Problem with Regular Callbacks

```ruby
class Order < ApplicationRecord
  after_save :send_confirmation_email  # DANGER!

  private

  def send_confirmation_email
    OrderMailer.confirmation(self).deliver_later
  end
end

# Problem scenario:
Order.transaction do
  order = Order.create!(...)  # Email sent!
  raise "Something went wrong"  # Transaction rolls back
  # But email was already sent for a non-existent order!
end
```

## after_commit to the Rescue

```ruby
class Order < ApplicationRecord
  after_commit :send_confirmation_email, on: :create

  private

  def send_confirmation_email
    # Only runs after transaction commits successfully
    OrderMailer.confirmation(self).deliver_later
  end
end
```

## Transaction Callback Types

```ruby
class Article < ApplicationRecord
  # Runs after successful commit
  after_commit :do_something

  # Runs after rollback
  after_rollback :handle_failure

  # Specific to actions
  after_create_commit :on_created     # Only after create commits
  after_update_commit :on_updated     # Only after update commits
  after_destroy_commit :on_destroyed  # Only after destroy commits
  after_save_commit :on_saved         # After create or update commits
end
```

## Common Use Cases

### Sending Emails

```ruby
class User < ApplicationRecord
  after_create_commit :send_welcome_email
  after_update_commit :send_email_changed_notification,
                      if: :saved_change_to_email?

  private

  def send_welcome_email
    UserMailer.welcome(self).deliver_later
  end

  def send_email_changed_notification
    UserMailer.email_changed(self, email_before_last_save).deliver_later
  end
end
```

### Background Jobs

```ruby
class Article < ApplicationRecord
  after_save_commit :reindex_in_search
  after_destroy_commit :remove_from_search

  private

  def reindex_in_search
    SearchIndexJob.perform_later(self)
  end

  def remove_from_search
    SearchRemoveJob.perform_later(id, self.class.name)
  end
end
```

### Cache Invalidation

```ruby
class Product < ApplicationRecord
  after_commit :invalidate_cache

  private

  def invalidate_cache
    Rails.cache.delete("product_#{id}")
    Rails.cache.delete("products_list")
    Rails.cache.delete("category_#{category_id}_products")
  end
end
```

### Broadcasting Updates

```ruby
class Message < ApplicationRecord
  after_create_commit :broadcast_message

  private

  def broadcast_message
    ActionCable.server.broadcast(
      "room_#{room_id}",
      { type: "new_message", message: as_json }
    )
  end
end
```

## Handling Rollbacks

```ruby
class Payment < ApplicationRecord
  after_rollback :log_failure
  after_rollback :notify_admin, on: :create

  private

  def log_failure
    Rails.logger.error "Payment #{id} transaction rolled back"
  end

  def notify_admin
    AdminMailer.payment_failed(self).deliver_later
  end
end
```

## Best Practices

1. **Use after_commit for external services** (emails, jobs, APIs)
2. **Use after_save for same-transaction side effects**
3. **Always use after_destroy_commit** when cleaning up external resources
4. **Be aware that ID exists** - after_commit has access to the saved ID

## Resources

- [Active Record Callbacks - Transaction Callbacks](https://guides.rubyonrails.org/active_record_callbacks.html#transaction-callbacks) — Guide to transaction callbacks

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*