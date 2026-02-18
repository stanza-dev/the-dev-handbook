---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-service-objects"
---

# Service Objects and Form Objects

Keep models thin by extracting complex business logic into service objects and form handling into form objects.

## Service Objects

Encapsulate complex operations that involve multiple models or external services:

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
  rescue InventoryError => e
    @order.errors.add(:base, "Some items are out of stock")
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
    SlackNotifier.new_order(@order)
  end
end

# Usage in controller
class CheckoutsController < ApplicationController
  def create
    @order = current_cart.to_order

    result = OrderCheckoutService.new(@order, payment_params).call

    if result
      redirect_to order_path(@order), notice: "Order placed!"
    else
      render :new
    end
  end
end
```

## Form Objects

Handle complex forms that don't map directly to a single model:

```ruby
# app/forms/registration_form.rb
class RegistrationForm
  include ActiveModel::Model
  include ActiveModel::Attributes

  attribute :email, :string
  attribute :password, :string
  attribute :password_confirmation, :string
  attribute :company_name, :string
  attribute :terms_accepted, :boolean

  validates :email, presence: true, format: { with: URI::MailTo::EMAIL_REGEXP }
  validates :password, presence: true, length: { minimum: 8 }
  validates :password_confirmation, presence: true
  validates :company_name, presence: true
  validates :terms_accepted, acceptance: true

  validate :passwords_match
  validate :email_not_taken

  def save
    return false unless valid?

    ActiveRecord::Base.transaction do
      @company = Company.create!(name: company_name)
      @user = User.create!(
        email: email,
        password: password,
        company: @company,
        role: "admin"
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

  private

  def passwords_match
    if password != password_confirmation
      errors.add(:password_confirmation, "doesn't match password")
    end
  end

  def email_not_taken
    if User.exists?(email: email)
      errors.add(:email, "is already taken")
    end
  end
end

# Usage in controller
class RegistrationsController < ApplicationController
  def new
    @form = RegistrationForm.new
  end

  def create
    @form = RegistrationForm.new(registration_params)

    if @form.save
      sign_in @form.user
      redirect_to dashboard_path
    else
      render :new
    end
  end

  private

  def registration_params
    params.require(:registration_form).permit(
      :email, :password, :password_confirmation,
      :company_name, :terms_accepted
    )
  end
end
```

## When to Use Each Pattern

| Pattern | Use When |
|---------|----------|
| **Scope** | Simple, reusable query conditions |
| **Query Object** | Complex queries with many parameters |
| **Service Object** | Multi-step operations, external services |
| **Form Object** | Forms spanning multiple models |

## Resources

- [Active Model Basics](https://guides.rubyonrails.org/active_model_basics.html) — Use Active Model in non-database classes

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*