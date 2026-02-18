---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-tdd-model-development"
---

# TDD Model Development

TDD works naturally with model development. Let's build a `Subscription` model using TDD.

## Define Requirements as Tests

```ruby
class SubscriptionTest < ActiveSupport::TestCase
  # Validation tests (write these first)
  test 'requires user' do
    sub = Subscription.new(plan: 'basic')
    assert_not sub.valid?
  end
  
  test 'requires plan' do
    sub = Subscription.new(user: users(:david))
    assert_not sub.valid?
  end
  
  test 'plan must be valid option' do
    sub = Subscription.new(
      user: users(:david),
      plan: 'invalid'
    )
    assert_not sub.valid?
  end
end
```

Run tests - all fail. Implement:

```ruby
class Subscription < ApplicationRecord
  belongs_to :user
  
  validates :plan, presence: true,
    inclusion: { in: %w[basic pro enterprise] }
end
```

## Add Behavior Tests

```ruby
test 'active? returns true for non-expired subscription' do
  sub = Subscription.new(expires_at: 1.day.from_now)
  assert sub.active?
end

test 'active? returns false for expired subscription' do
  sub = Subscription.new(expires_at: 1.day.ago)
  assert_not sub.active?
end

test 'days_remaining calculates correctly' do
  sub = Subscription.new(expires_at: 10.days.from_now)
  assert_equal 10, sub.days_remaining
end
```

Implement:

```ruby
class Subscription < ApplicationRecord
  def active?
    expires_at.present? && expires_at > Time.current
  end
  
  def days_remaining
    return 0 unless active?
    ((expires_at - Time.current) / 1.day).ceil
  end
end
```

## Test Complex Business Logic

```ruby
test 'renew extends expiration by plan duration' do
  travel_to Time.zone.local(2024, 1, 1) do
    sub = subscriptions(:basic_active)
    original_expiry = sub.expires_at
    
    sub.renew!
    
    # Basic plan adds 30 days
    assert_equal original_expiry + 30.days, sub.expires_at
  end
end

test 'upgrade changes plan and adjusts billing' do
  sub = subscriptions(:basic_active)
  
  sub.upgrade_to('pro')
  
  assert_equal 'pro', sub.plan
  assert sub.prorated_credit.positive?
end
```

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*