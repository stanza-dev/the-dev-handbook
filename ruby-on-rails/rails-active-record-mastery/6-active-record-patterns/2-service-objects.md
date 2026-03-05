---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-service-objects"
---

# Service Objects and Form Objects

## Introduction
As Rails applications grow, models accumulate business logic that does not belong there. Service objects encapsulate multi-step operations involving multiple models or external services. Form objects handle complex forms that span multiple models. Both patterns keep your models focused on data access and validation.

## Key Concepts
- **Service object**: A plain Ruby class that encapsulates a single business operation (e.g., checkout, registration, import).
- **Form object**: A class that includes `ActiveModel::Model` to handle form data that spans multiple models.
- **Single Responsibility**: Each service object does one thing. Name it as a verb: `CreateOrder`, `ProcessPayment`, `ImportUsers`.
- **`ActiveModel::Model`**: A mixin that gives plain classes validation, error handling, and form builder compatibility.

## Real World Context
A checkout process touches orders, payments, inventory, notifications, and analytics. Putting all this in `Order#save` callbacks makes the model untestable and fragile. A `CheckoutService` class is explicit, testable, and easy to reason about. Every mature Rails codebase uses service objects for complex operations.

## Deep Dive

### Service Objects

```ruby
# app/services/order_checkout_service.rb
class OrderCheckoutService
  def initialize(order, payment_params)
    @order = order
    @payment_params = payment_params
  end

  def call
    ActiveRecord::Base.transaction do
      validate_inventory!
      process_payment!
      create_shipment!
      send_notifications
      @order
    end
  rescue PaymentError => e
    @order.errors.add(:base, e.message)
    false
  end

  private

  def validate_inventory!
    @order.line_items.each do |item|
      unless item.product.in_stock?(item.quantity)
        raise InventoryError, "#{item.product.name} is out of stock"
      end
    end
  end

  def process_payment!
    payment = PaymentGateway.charge(
      amount: @order.total,
      card: @payment_params
    )
    unless payment.success?
      raise PaymentError, payment.error_message
    end
    @order.update!(payment_id: payment.id, status: "paid")
  end

  def create_shipment!
    @order.create_shipment!(
      address: @order.shipping_address,
      items: @order.line_items
    )
  end

  def send_notifications
    OrderMailer.confirmation(@order).deliver_later
  end
end

# Usage in controller
result = OrderCheckoutService.new(@order, payment_params).call
```

The service wraps everything in a transaction. If any step fails, the entire operation rolls back.

### Form Objects

```ruby
# app/forms/registration_form.rb
class RegistrationForm
  include ActiveModel::Model
  include ActiveModel::Attributes

  attribute :email, :string
  attribute :password, :string
  attribute :company_name, :string
  attribute :terms_accepted, :boolean

  validates :email, presence: true, format: { with: URI::MailTo::EMAIL_REGEXP }
  validates :password, presence: true, length: { minimum: 8 }
  validates :company_name, presence: true
  validates :terms_accepted, acceptance: true

  def save
    return false unless valid?

    ActiveRecord::Base.transaction do
      @company = Company.create!(name: company_name)
      @user = User.create!(
        email: email, password: password,
        company: @company, role: "admin"
      )
    end
    true
  rescue ActiveRecord::RecordInvalid => e
    errors.add(:base, e.message)
    false
  end

  def user
    @user
  end
end
```

Form objects validate all inputs before touching the database, and they work seamlessly with Rails form builders.

### When to Use Each Pattern

| Pattern | Use When |
|---------|----------|
| **Scope** | Simple, reusable query conditions |
| **Query Object** | Complex queries with many parameters |
| **Service Object** | Multi-step operations, external services |
| **Form Object** | Forms spanning multiple models |

## Common Pitfalls
1. **Service objects that do too much** -- A service should do one operation. If it grows beyond 100 lines, split it.
2. **Skipping validations in service objects** -- Always validate inputs before executing. Use `valid?` or form objects.
3. **Not wrapping in transactions** -- Multi-model operations must be wrapped in `ActiveRecord::Base.transaction` to ensure atomicity.

## Best Practices
1. **Name services as verbs** -- `ProcessPayment`, `SendInvitation`, `ImportCSV` clearly communicate intent.
2. **Return truthy/falsy or a result object** -- Consistent return values make controller logic simple.
3. **Keep the `call` method short** -- It should read like a table of contents. Extract steps into private methods.

## Summary
- Service objects encapsulate multi-step business operations.
- Form objects handle forms that span multiple models.
- Both patterns keep models thin and controllers simple.
- Wrap multi-model operations in transactions.
- Name services as verbs and keep the `call` method short.

## Code Examples

**A form object using ActiveModel::Model -- validates inputs and creates multiple records in a transaction**

```ruby
class RegistrationForm
  include ActiveModel::Model
  include ActiveModel::Attributes

  attribute :email, :string
  attribute :password, :string
  attribute :company_name, :string

  validates :email, presence: true
  validates :password, length: { minimum: 8 }

  def save
    return false unless valid?
    ActiveRecord::Base.transaction do
      @company = Company.create!(name: company_name)
      @user = User.create!(email: email, password: password, company: @company)
    end
    true
  end
end
```


## Resources

- [Active Model Basics](https://guides.rubyonrails.org/active_model_basics.html) — Use Active Model in non-database classes

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*