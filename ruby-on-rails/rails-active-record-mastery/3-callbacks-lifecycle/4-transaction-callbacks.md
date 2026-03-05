---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-transaction-callbacks"
---

# Transaction Callbacks

## Introduction
Transaction callbacks run after the database transaction is committed or rolled back. They are crucial for side effects that should only happen if the save actually persisted to the database. If you have ever sent an email for a record that was rolled back, you understand why these callbacks exist.

## Key Concepts
- **`after_commit`**: Runs only after the wrapping database transaction has been successfully committed.
- **`after_rollback`**: Runs only after the wrapping transaction has been rolled back.
- **`after_create_commit` / `after_update_commit` / `after_destroy_commit`**: Shorthand for `after_commit` scoped to a specific action.
- **`after_save_commit`**: Runs after commit for both create and update actions.

## Real World Context
In production, database operations often happen inside transactions. A checkout process might create an order, update inventory, and charge a payment. If the payment fails, the transaction rolls back. If you sent a confirmation email in `after_save`, the user receives a confirmation for an order that does not exist. `after_commit` prevents this class of bugs.

## Deep Dive

### The Problem with after_save

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
  order = Order.create!(...)  # Email enqueued!
  raise "Something went wrong"  # Transaction rolls back
  # But email was already enqueued for a non-existent order!
end
```

The email is enqueued during `after_save`, which runs inside the transaction. When the transaction rolls back, the order disappears, but the email job is already in the queue.

### after_commit to the Rescue

```ruby
class Order < ApplicationRecord
  after_commit :send_confirmation_email, on: :create

  private

  def send_confirmation_email
    OrderMailer.confirmation(self).deliver_later
  end
end
```

Now the email is only enqueued after the transaction commits successfully. If the transaction rolls back, the callback never fires.

### Action-Specific Shortcuts

```ruby
class Article < ApplicationRecord
  after_create_commit :on_created
  after_update_commit :on_updated
  after_destroy_commit :on_destroyed
  after_save_commit :on_saved  # create or update
end
```

These are syntactic sugar for `after_commit :method, on: :create` etc.

### Handling Rollbacks

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

### Cache Invalidation

```ruby
class Product < ApplicationRecord
  after_commit :invalidate_cache

  private

  def invalidate_cache
    Rails.cache.delete("product_#{id}")
    Rails.cache.delete("products_list")
  end
end
```

Cache invalidation is a perfect use case for `after_commit` because you want the cache to reflect the committed state.

## Common Pitfalls
1. **Using after_save for external services** -- Emails, background jobs, webhooks, and API calls should always use `after_commit`. The transaction could roll back after `after_save` fires.
2. **Forgetting that after_commit runs outside the transaction** -- Any database operations inside `after_commit` run in a new, separate transaction.
3. **Multiple after_commit on the same action** -- If you define multiple `after_update_commit` callbacks, they all run. Make sure they are independent and idempotent.

## Best Practices
1. **Use `after_commit` for all external side effects** -- emails, jobs, API calls, WebSocket broadcasts.
2. **Use `after_save` only for same-transaction side effects** -- like updating related records that should roll back together.
3. **Use `after_destroy_commit` when cleaning up external resources** -- file storage, search indexes, CDN cache.

## Summary
- `after_commit` runs only after a successful transaction commit.
- `after_rollback` runs only after a transaction rollback.
- Use `after_create_commit`, `after_update_commit`, `after_destroy_commit` for action-specific hooks.
- Always use `after_commit` (not `after_save`) for emails, background jobs, and external API calls.
- `after_save` runs inside the transaction and can fire for operations that later roll back.

## Code Examples

**Using after_commit instead of after_save for side effects -- ensures the email is only sent after the transaction succeeds**

```ruby
class Order < ApplicationRecord
  # WRONG: runs inside transaction, may fire for rolled-back records
  # after_save :send_confirmation_email

  # CORRECT: runs only after successful commit
  after_create_commit :send_confirmation_email
  after_update_commit :clear_cache
  after_destroy_commit :cleanup_files

  private

  def send_confirmation_email
    OrderMailer.confirmation(self).deliver_later
  end
end
```


## Resources

- [Active Record Callbacks - Transaction Callbacks](https://guides.rubyonrails.org/active_record_callbacks.html#transaction-callbacks) — Guide to transaction callbacks

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*