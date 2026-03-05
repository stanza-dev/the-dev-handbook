---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-validation-contexts"
---

# Validation Contexts and Conditional Logic

## Introduction
Not every validation should run every time. A password is required on registration but not when updating a profile. A shipping address is required for physical orders but not digital ones. Validation contexts and conditional logic let you control exactly when each validation fires.

## Key Concepts
- **Validation context**: A named scenario (`:create`, `:update`, or custom) that controls when validations run.
- **`:if` / `:unless` options**: Conditionally enable or disable a validation based on a predicate method or lambda.
- **`with_options`**: Groups multiple validations under the same condition to reduce repetition.
- **Strict validations**: Raise exceptions instead of adding errors, used for invariants that should never be violated.

## Real World Context
E-commerce apps validate shipping addresses only for physical products. SaaS platforms require terms acceptance on signup but not on profile edits. Admin users have stricter validations than regular users. Conditional validations let you model these real business rules without resorting to complex if/else logic.

## Deep Dive

### Validation Contexts

```ruby
class User < ApplicationRecord
  validates :email, presence: true
  validates :password, presence: true, on: :create
  validates :terms_accepted, acceptance: true, on: :create
  validates :phone, presence: true, on: :phone_verification
end

user = User.new(email: "test@test.com")
user.valid?                     # true (password not checked)
user.valid?(:create)            # false (password required)
user.valid?(:phone_verification) # checks phone
```

The `:create` and `:update` contexts are applied automatically by `save`. Custom contexts like `:phone_verification` must be triggered explicitly.

### Conditional Validations

```ruby
class Order < ApplicationRecord
  validates :shipping_address, presence: true, unless: :digital_only?
  validates :card_number, presence: true, if: :paying_with_card?

  # With lambda
  validates :gift_message, length: { maximum: 500 }, if: -> { gift? }

  # Multiple conditions (all must pass)
  validates :delivery_instructions, presence: true,
    if: [:requires_delivery?, :expedited?]
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

All three validations share the `if: :admin?` condition without repeating it.

### Strict Validations

```ruby
class Account < ApplicationRecord
  validates :balance, numericality: {
    greater_than_or_equal_to: 0, strict: true
  }
end

# Raises ActiveModel::StrictValidationFailed instead of adding error
```

Use strict validations for invariants that indicate a bug in your code, not user input errors.

### Database-Level Constraints

Validations run in Ruby and can be bypassed. Always complement important validations with database constraints:

```ruby
class AddConstraintsToUsers < ActiveRecord::Migration[8.1]
  def change
    change_column_null :users, :email, false
    add_index :users, :email, unique: true
    add_check_constraint :users, "balance >= 0", name: "positive_balance"
  end
end
```

## Common Pitfalls
1. **Relying only on model validations** -- Validations can be bypassed with `update_column`, `save(validate: false)`, or raw SQL. Add database constraints for critical rules.
2. **Overloading the :create context** -- If you add too many `:create`-only validations, the initial save becomes brittle. Consider whether the validation truly applies only to creation.
3. **Nested conditionals** -- Avoid `if: -> { condition_a? && (condition_b? || condition_c?) }`. Extract complex conditions into a named method for readability.

## Best Practices
1. **Use contexts sparingly** -- `:create` and `:update` cover most cases. Custom contexts add complexity.
2. **Mirror validations with database constraints** -- For uniqueness, NOT NULL, and range checks, always add the corresponding database constraint.
3. **Use `with_options` for role-based validations** -- Grouping admin-only or premium-only validations improves readability.

## Summary
- Use `on: :create` or `on: :update` to scope validations to specific actions.
- Use `:if` and `:unless` with symbols, lambdas, or arrays for conditional validations.
- `with_options` groups multiple validations under the same condition.
- Strict validations raise exceptions for invariants.
- Always back important validations with database constraints.

## Code Examples

**Conditional validations using :if, :unless, and with_options -- different rules apply based on order type**

```ruby
class Order < ApplicationRecord
  validates :shipping_address, presence: true, unless: :digital_only?
  validates :card_number, presence: true, if: :paying_with_card?

  with_options if: :expedited? do |exp|
    exp.validates :delivery_date, presence: true
    exp.validates :phone, presence: true
  end

  private

  def digital_only?
    line_items.all?(&:digital?)
  end
end
```


## Resources

- [Active Record Validations - Conditional](https://guides.rubyonrails.org/active_record_validations.html#conditional-validation) — Guide to conditional validations

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*