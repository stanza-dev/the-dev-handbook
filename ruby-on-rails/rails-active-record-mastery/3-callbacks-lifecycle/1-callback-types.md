---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-callback-types"
---

# Understanding Callbacks

Callbacks are methods that get called at specific points in an object's lifecycle. They let you trigger logic during creation, updates, saves, and deletion.

## The Callback Chain

When you save a record, Rails runs callbacks in this order:

### Creating a New Record

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

### Updating an Existing Record

```
before_validation
after_validation
before_save
around_save
before_update
around_update
[UPDATE database]
after_update
after_save
after_commit / after_rollback
```

### Destroying a Record

```
before_destroy
around_destroy
[DELETE FROM database]
after_destroy
after_commit / after_rollback
```

## Defining Callbacks

```ruby
class Article < ApplicationRecord
  # Method reference
  before_save :set_published_at
  after_create :notify_subscribers

  # Block syntax
  before_validation { self.title = title.strip if title }

  # Lambda syntax
  after_save ->(article) { Rails.logger.info "Saved: #{article.title}" }

  private

  def set_published_at
    self.published_at ||= Time.current if published?
  end

  def notify_subscribers
    NotificationJob.perform_later(self)
  end
end
```

## Common Callback Use Cases

### Before Validation - Normalizing Data

```ruby
class User < ApplicationRecord
  before_validation :normalize_email

  private

  def normalize_email
    self.email = email.downcase.strip if email
  end
end
```

### Before Save - Setting Defaults

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

### After Create - Side Effects

```ruby
class Order < ApplicationRecord
  after_create :send_confirmation_email
  after_create :update_inventory

  private

  def send_confirmation_email
    OrderMailer.confirmation(self).deliver_later
  end

  def update_inventory
    line_items.each do |item|
      item.product.decrement!(:stock, item.quantity)
    end
  end
end
```

### Before Destroy - Cleanup

```ruby
class User < ApplicationRecord
  before_destroy :check_for_orders
  before_destroy :cancel_subscriptions

  private

  def check_for_orders
    if orders.any?
      errors.add(:base, "Cannot delete user with orders")
      throw(:abort)  # Prevents deletion
    end
  end

  def cancel_subscriptions
    subscriptions.each(&:cancel!)
  end
end
```

## Halting the Callback Chain

```ruby
class Article < ApplicationRecord
  before_save :check_spam

  private

  def check_spam
    if SpamDetector.spam?(body)
      errors.add(:body, "contains spam")
      throw(:abort)  # Stops save and other callbacks
    end
  end
end
```

## Resources

- [Active Record Callbacks](https://guides.rubyonrails.org/active_record_callbacks.html) — Complete guide to Active Record callbacks

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*