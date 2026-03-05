---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-conditional-callbacks"
---

# Conditional Callbacks

## Introduction
Callbacks can be conditionally executed using `:if` and `:unless` options. This gives you fine-grained control over when callbacks run, preventing unnecessary work and keeping your models efficient.

## Key Concepts
- **`:if` option**: The callback runs only when the condition returns true.
- **`:unless` option**: The callback runs only when the condition returns false.
- **Callback classes**: Reusable callback logic extracted into dedicated classes.
- **`on:` option**: Limits callbacks to specific actions (`:create`, `:update`, `:destroy`).

## Real World Context
In production, you rarely want every callback to run every time. An order notification should only fire when the status changes to "shipped", not on every save. An audit log should only record changes, not no-op saves. Conditional callbacks keep your app efficient and prevent unintended side effects.

## Deep Dive

### Using :if and :unless with Symbols

```ruby
class Order < ApplicationRecord
  after_save :send_notification, if: :status_changed_to_shipped?
  before_destroy :check_refundable, unless: :admin_user?

  private

  def status_changed_to_shipped?
    saved_change_to_status? && status == "shipped"
  end

  def admin_user?
    Current.user&.admin?
  end
end
```

The symbol form calls a method on the record. This is the most readable approach.

### Using Procs/Lambdas

```ruby
class Article < ApplicationRecord
  after_save :notify_subscribers,
    if: -> { published? && saved_change_to_published? }

  after_create :celebrate,
    if: -> { featured? && published? && author.verified? }
end
```

Lambdas are convenient for short inline conditions without defining a separate method.

### Multiple Conditions

```ruby
class User < ApplicationRecord
  # Both conditions must be true (AND logic)
  after_save :send_welcome_email,
    if: :email_changed?,
    unless: :skip_email_notification?

  # Array of conditions (all must pass)
  before_save :validate_premium_features,
    if: [:premium?, :features_changed?]
end
```

### Grouping with with_options

```ruby
class User < ApplicationRecord
  with_options if: :admin? do |admin|
    admin.validates :admin_code, presence: true
    admin.validates :department, presence: true
    admin.validates :clearance_level, inclusion: { in: 1..5 }
  end
end
```

The `with_options` method applies the same condition to multiple validations or callbacks, reducing repetition.

### Callback Classes

For complex or reusable callback logic, extract to a dedicated class:

```ruby
# app/callbacks/audit_callback.rb
class AuditCallback
  def after_create(record)
    AuditLog.create!(
      action: "create",
      auditable: record,
      user: Current.user,
      changes: record.saved_changes
    )
  end

  def after_update(record)
    return unless record.saved_changes.any?
    AuditLog.create!(
      action: "update",
      auditable: record,
      user: Current.user,
      changes: record.saved_changes
    )
  end
end

# Usage in model
class Article < ApplicationRecord
  after_create AuditCallback.new
  after_update AuditCallback.new
end
```

### Skipping Callbacks

```ruby
# update_column skips all callbacks and validations
article.update_column(:views_count, article.views_count + 1)

# update_columns for multiple columns
article.update_columns(views_count: 100, last_viewed_at: Time.current)
```

Use these sparingly and only for performance-critical operations like incrementing counters.

## Common Pitfalls
1. **Forgetting `saved_change_to_*` vs `*_changed?`** -- Inside `after_save`, use `saved_change_to_status?` (past tense). Inside `before_save`, use `status_changed?` (present tense).
2. **Over-using `update_column`** -- It skips validations and callbacks, which can leave your data in an inconsistent state. Only use it when you explicitly need to bypass the lifecycle.
3. **Condition methods with side effects** -- The `:if` method should be a pure predicate (return true/false). Never put logic that modifies state in a condition method.

## Best Practices
1. **Use `on:` to scope to specific actions** -- `after_commit :reindex, on: [:create, :update]` is clearer than a conditional that checks `new_record?`.
2. **Extract complex conditions to named methods** -- `if: :should_send_notification?` is more readable than a multi-line lambda.
3. **Use callback classes for cross-cutting concerns** -- Audit logging, cache invalidation, and search indexing are good candidates.

## Summary
- Use `:if` and `:unless` to control when callbacks run.
- Conditions can be symbols (method names), procs, or arrays of conditions.
- `with_options` groups multiple validations/callbacks under the same condition.
- Callback classes encapsulate reusable cross-cutting concerns.
- Use `saved_change_to_*?` in `after_save` and `*_changed?` in `before_save`.

## Code Examples

**Conditional callbacks -- using :if with a predicate method and on: to scope to specific lifecycle events**

```ruby
class Order < ApplicationRecord
  # Only notify when status changes to shipped
  after_save :send_shipping_notification,
    if: :status_changed_to_shipped?

  # Scope callback to specific actions
  after_commit :reindex_search, on: [:create, :update]

  private

  def status_changed_to_shipped?
    saved_change_to_status? && status == "shipped"
  end
end
```


## Resources

- [Active Record Callbacks - Conditional](https://guides.rubyonrails.org/active_record_callbacks.html#conditional-callbacks) — Guide to conditional callbacks

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*