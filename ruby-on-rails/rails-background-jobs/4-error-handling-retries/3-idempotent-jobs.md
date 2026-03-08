---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-idempotent-jobs"
---

# Designing Idempotent Jobs

## Introduction
An idempotent job produces the same result whether it runs once or ten times. Since jobs can be retried, idempotency is a requirement for correctness.

## Key Concepts
- **Idempotency**: Running an operation multiple times has the same effect as once.
- **Idempotency Key**: A unique identifier to detect duplicate processing.
- **Check-then-act**: Check if work was done before doing it.

## Real World Context
A job that charges a credit card: if it succeeds but the worker crashes before marking it complete, the retry charges again. Idempotent design prevents this.

## Deep Dive

### Check Before Acting

```ruby
class SendWelcomeEmailJob < ApplicationJob
  def perform(user_id)
    user = User.find_by(id: user_id)
    return unless user
    return if user.welcome_email_sent?

    UserMailer.welcome(user).deliver_now
    user.update!(welcome_email_sent: true)
  end
end
```

### Idempotency Keys for APIs

```ruby
class ChargePaymentJob < ApplicationJob
  def perform(order_id)
    order = Order.find(order_id)
    return if order.paid?

    PaymentGateway.charge(
      amount: order.total,
      idempotency_key: "order_#{order.id}"
    )
    order.update!(status: :paid)
  end
end
```

### Database Constraints

```ruby
class CreateInvoiceJob < ApplicationJob
  def perform(order_id, month)
    Invoice.create!(order_id: order_id, month: month, amount: calculate(order_id, month))
  rescue ActiveRecord::RecordNotUnique
    Rails.logger.info "Invoice already exists"
  end
end
```

## Common Pitfalls
1. **Relying on in-memory state** — Always check persistent storage.
2. **Non-atomic check-then-act** — Use database constraints for race-condition-free checks.

## Best Practices
1. **Design every job to be idempotent from day one.**
2. **Use unique database constraints for critical operations.**

## Summary
- Idempotent jobs produce the same result regardless of run count.
- Use check-before-act patterns with database flags.
- Use idempotency keys for external API calls.
- Database unique constraints provide atomic duplicate prevention.

## Code Examples

**Idempotent provisioning — checks a flag before doing work**

```ruby
class ProvisionAccountJob < ApplicationJob
  def perform(signup_id)
    signup = Signup.find(signup_id)
    return if signup.provisioned?

    account = Account.create!(owner: signup.user)
    signup.update!(provisioned: true, account_id: account.id)
  end
end
```


## Resources

- [Active Job Basics](https://guides.rubyonrails.org/active_job_basics.html) — Official Active Job guide

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*