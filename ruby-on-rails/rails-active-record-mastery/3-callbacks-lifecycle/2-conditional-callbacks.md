---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-conditional-callbacks"
---

# Conditional Callbacks

Callbacks can be conditionally executed using `:if` and `:unless` options. This gives you fine-grained control over when callbacks run.

## Using :if and :unless

### With a Symbol (Method Name)

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

### With a Proc/Lambda

```ruby
class Article < ApplicationRecord
  after_save :notify_subscribers, if: -> { published? && saved_change_to_published? }

  before_validation :set_author, unless: -> { author.present? }

  # Multiple conditions with lambda
  after_create :celebrate,
    if: -> { featured? && published? && author.verified? }
end
```

### Multiple Conditions

```ruby
class User < ApplicationRecord
  # Both conditions must be true
  after_save :send_welcome_email,
    if: :email_changed?,
    unless: :skip_email_notification?

  # Using arrays (all must pass)
  before_save :validate_premium_features,
    if: [:premium?, :features_changed?]
end
```

## Callback Classes

For complex or reusable callback logic, use callback classes:

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

  def after_destroy(record)
    AuditLog.create!(
      action: "destroy",
      auditable: record,
      user: Current.user
    )
  end
end

# Usage in model
class Article < ApplicationRecord
  after_create AuditCallback.new
  after_update AuditCallback.new
  after_destroy AuditCallback.new
end
```

## On-Condition Callbacks

Limit callbacks to specific actions:

```ruby
class Article < ApplicationRecord
  # Only on create
  after_commit :notify_author, on: :create

  # Only on update
  after_commit :clear_cache, on: :update

  # Only on destroy
  after_commit :cleanup_files, on: :destroy

  # Multiple actions
  after_commit :reindex_search, on: [:create, :update]
end
```

## Tracking Changes in Callbacks

```ruby
class Article < ApplicationRecord
  after_save :handle_status_change

  private

  def handle_status_change
    # Check if specific attribute changed
    if saved_change_to_status?
      old_status, new_status = saved_change_to_status
      Rails.logger.info "Status changed from #{old_status} to #{new_status}"

      if new_status == "published"
        notify_subscribers
      end
    end

    # Check what changed
    if saved_changes.key?("title")
      update_slug
    end
  end
end
```

## Skip Callbacks

Sometimes you need to skip callbacks:

```ruby
# Using update_column (skips all callbacks and validations)
article.update_column(:views_count, article.views_count + 1)

# Using update_columns
article.update_columns(views_count: 100, last_viewed_at: Time.current)

# In tests or special cases
Article.skip_callback(:save, :after, :send_notification) do
  article.save!
end
```

## Resources

- [Active Record Callbacks - Conditional](https://guides.rubyonrails.org/active_record_callbacks.html#conditional-callbacks) — Guide to conditional callbacks

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*