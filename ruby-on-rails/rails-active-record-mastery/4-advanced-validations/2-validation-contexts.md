---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-validation-contexts"
---

# Validation Contexts and Conditional Logic

Control when validations run using contexts, conditions, and grouped validations.

## Validation Contexts

Run different validations in different situations:

```ruby
class User < ApplicationRecord
  validates :email, presence: true
  validates :password, presence: true, on: :create
  validates :terms_accepted, acceptance: true, on: :create

  # Custom context
  validates :phone, presence: true, on: :phone_verification
  validates :address, presence: true, on: :checkout
end

# Usage
user = User.new(email: "test@test.com")
user.valid?                     # true (password not required for general validation)
user.valid?(:create)            # false (password required for create)
user.save                       # Uses :create context

# Custom context
user.valid?(:phone_verification)  # Checks phone validation
user.valid?(:checkout)            # Checks address validation
```

## Conditional Validations

### With :if and :unless

```ruby
class Order < ApplicationRecord
  validates :shipping_address, presence: true, unless: :digital_only?
  validates :card_number, presence: true, if: :paying_with_card?

  # With lambda
  validates :gift_message, length: { maximum: 500 }, if: -> { gift? }

  # Multiple conditions (must pass all)
  validates :delivery_instructions, presence: true,
    if: [:requires_delivery?, :expedited?]

  private

  def digital_only?
    line_items.all?(&:digital?)
  end

  def paying_with_card?
    payment_method == "card"
  end
end
```

### Grouping Conditional Validations

```ruby
class User < ApplicationRecord
  with_options if: :admin? do |admin|
    admin.validates :admin_code, presence: true
    admin.validates :department, presence: true
    admin.validates :clearance_level, inclusion: { in: 1..5 }
  end

  with_options unless: :guest? do |user|
    user.validates :email, presence: true, uniqueness: true
    user.validates :password, length: { minimum: 8 }
  end
end
```

## Strict Validations

Raise exceptions instead of adding errors:

```ruby
class Account < ApplicationRecord
  validates :subdomain, uniqueness: { strict: true }
  validates :balance, numericality: { greater_than_or_equal_to: 0, strict: true }
end

# Raises ActiveModel::StrictValidationFailed
account.subdomain = existing_subdomain
account.valid?  # => raises exception
```

## Database-Level Constraints

Validations run in Ruby; also add database constraints:

```ruby
class AddConstraintsToUsers < ActiveRecord::Migration[8.0]
  def change
    # NOT NULL constraint
    change_column_null :users, :email, false

    # Unique constraint
    add_index :users, :email, unique: true

    # Check constraint
    add_check_constraint :users, "balance >= 0", name: "positive_balance"

    # Foreign key constraint
    add_foreign_key :orders, :users, on_delete: :cascade
  end
end
```

## Validation Error Details

```ruby
class Product < ApplicationRecord
  validates :price, numericality: {
    greater_than: 0,
    message: "must be positive (got %{value})"
  }

  validates :quantity, numericality: {
    less_than_or_equal_to: :max_quantity,
    message: "cannot exceed %{count}"
  }
end

# Access error details
product.valid?
product.errors[:price]           # => ["must be positive (got -5)"]
product.errors.details[:price]   # => [{error: :greater_than, value: -5, count: 0}]
```

## Resources

- [Active Record Validations - Conditional](https://guides.rubyonrails.org/active_record_validations.html#conditional-validation) — Guide to conditional validations

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*