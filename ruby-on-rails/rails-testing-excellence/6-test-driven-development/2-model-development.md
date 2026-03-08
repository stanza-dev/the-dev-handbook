---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-tdd-model-development"
---

# TDD for Model Development

## Introduction
Models are the ideal starting point for TDD in Rails. They contain your business logic, validations, and data transformations — all of which are pure functions with clear inputs and outputs. By writing model tests first, you define your domain rules before implementing them, producing models that are focused, well-validated, and free of speculative features.

## Key Concepts
- **Validation Tests**: Tests that assert a model rejects invalid data. Write these first to define your data integrity rules.
- **Behavior Tests**: Tests that verify model methods return the correct results. These define your business logic contract.
- **Edge Case Tests**: Tests that cover boundary conditions like nil values, empty strings, and extreme dates.
- **Fixtures**: YAML files in `test/fixtures/` that provide sample data for your tests. Rails loads them into the test database before each test.

## Real World Context
In a SaaS application, the `Subscription` model might need to know if it is active, calculate remaining days, handle renewal, and prorate upgrades. Without TDD, developers often implement these methods and then write tests that merely confirm the implementation. With TDD, you write the test first — "a subscription with an expiry in the past should not be active" — and the implementation follows naturally from the requirement.

## Deep Dive
Let us build a `Subscription` model using TDD, starting with validations and progressing to business logic.

### Step 1: Define Validation Rules as Tests
Before writing any model code, describe what valid and invalid data looks like:

```ruby
# test/models/subscription_test.rb
class SubscriptionTest < ActiveSupport::TestCase
  test 'requires user' do
    sub = Subscription.new(plan: 'basic')
    assert_not sub.valid?
    assert_includes sub.errors[:user], 'must exist'
  end

  test 'requires plan' do
    sub = Subscription.new(user: users(:david))
    assert_not sub.valid?
    assert_includes sub.errors[:plan], "can't be blank"
  end

  test 'plan must be a valid option' do
    sub = Subscription.new(user: users(:david), plan: 'invalid')
    assert_not sub.valid?
    assert_includes sub.errors[:plan], 'is not included in the list'
  end

  test 'valid with user and valid plan' do
    sub = Subscription.new(user: users(:david), plan: 'basic')
    assert sub.valid?
  end
end
```

Run the tests — all fail (Red). Now implement the validations:

```ruby
# app/models/subscription.rb
class Subscription < ApplicationRecord
  belongs_to :user

  validates :plan, presence: true,
    inclusion: { in: %w[basic pro enterprise] }
end
```

Run the tests again — all pass (Green).

### Step 2: Add Behavior Tests
With validations in place, test the methods your model needs:

```ruby
test 'active? returns true for non-expired subscription' do
  sub = Subscription.new(expires_at: 1.day.from_now)
  assert sub.active?
end

test 'active? returns false for expired subscription' do
  sub = Subscription.new(expires_at: 1.day.ago)
  assert_not sub.active?
end

test 'active? returns false when expires_at is nil' do
  sub = Subscription.new(expires_at: nil)
  assert_not sub.active?
end

test 'days_remaining calculates correctly' do
  travel_to Time.zone.local(2026, 3, 1) do
    sub = Subscription.new(expires_at: Time.zone.local(2026, 3, 11))
    assert_equal 10, sub.days_remaining
  end
end

test 'days_remaining returns 0 for expired subscription' do
  sub = Subscription.new(expires_at: 1.day.ago)
  assert_equal 0, sub.days_remaining
end
```

All fail (Red). Implement the methods:

```ruby
class Subscription < ApplicationRecord
  belongs_to :user

  validates :plan, presence: true,
    inclusion: { in: %w[basic pro enterprise] }

  def active?
    expires_at.present? && expires_at > Time.current
  end

  def days_remaining
    return 0 unless active?
    ((expires_at - Time.current) / 1.day).ceil
  end
end
```

All pass (Green).

### Step 3: Test Complex Business Logic
For operations that change state, test both the outcome and side effects:

```ruby
test 'renew! extends expiration by 30 days for basic plan' do
  travel_to Time.zone.local(2026, 1, 1) do
    sub = subscriptions(:basic_active)
    original_expiry = sub.expires_at

    sub.renew!

    assert_equal original_expiry + 30.days, sub.expires_at
  end
end

test 'upgrade_to changes plan and calculates prorated credit' do
  sub = subscriptions(:basic_active)

  sub.upgrade_to('pro')

  assert_equal 'pro', sub.plan
  assert sub.prorated_credit.positive?
end
```

These tests drive the implementation of `renew!` and `upgrade_to` — you write exactly the code needed to satisfy the test assertions.

## Common Pitfalls
1. **Testing implementation instead of behavior** — Do not test that a method calls another method internally. Test the observable outcome: "after calling `renew!`, `expires_at` is 30 days later."
2. **Forgetting edge cases in the Red phase** — Write tests for nil values, empty strings, and boundary dates before implementing. It is much harder to add edge case handling retroactively.
3. **Using `travel_to` without freezing time** — Time-dependent tests must use `travel_to` to produce deterministic results. Without it, tests that pass at noon may fail at midnight.

## Best Practices
1. **Start with validation tests** — Validations define your data contract. Writing them first ensures your model rejects bad data before you add any behavior.
2. **Test one behavior per test** — Each test should assert a single outcome. "Subscription is active and has 10 days remaining" should be two separate tests.
3. **Use `travel_to` for all time-dependent logic** — Wrap time-sensitive assertions in `travel_to` blocks to freeze time and eliminate flakiness.

## Summary
- Start model TDD with validation tests — they define what valid data looks like.
- Progress to behavior tests that verify method return values and state changes.
- Use `travel_to` to freeze time in tests that depend on dates or durations.
- Test observable outcomes, not internal implementation details.
- Each test should assert a single behavior, keeping failures easy to diagnose.

## Code Examples

**The three stages of model TDD — validation tests first, then behavior tests, then complex logic tests with time freezing**

```ruby
# TDD workflow for a Subscription model
# 1. Write validation tests first
test 'requires plan' do
  sub = Subscription.new(user: users(:david))
  assert_not sub.valid?
end

# 2. Write behavior tests
test 'active? returns false for expired subscription' do
  sub = Subscription.new(expires_at: 1.day.ago)
  assert_not sub.active?
end

# 3. Write complex logic tests with time freezing
test 'renew! extends expiration by 30 days' do
  travel_to Time.zone.local(2026, 1, 1) do
    sub = subscriptions(:basic_active)
    sub.renew!
    assert_equal sub.expires_at, Time.zone.local(2026, 1, 1) + 30.days
  end
end
```


## Resources

- [Rails Testing Guide — Model Testing](https://guides.rubyonrails.org/testing.html#model-testing) — Official Rails guide section on testing Active Record models, validations, and callbacks

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*