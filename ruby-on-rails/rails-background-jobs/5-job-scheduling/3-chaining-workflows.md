---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-job-chaining-workflows"
---

# Job Chaining and Workflows

## Introduction
Complex processes involve multiple steps in sequence: validate, charge, fulfill, notify. Job chaining breaks these into focused, independently retriable jobs.

## Key Concepts
- **Job Chain**: Each job triggers the next upon completion.
- **State Machine**: Model status columns coordinate between steps.

## Real World Context
An e-commerce order: validation, payment, inventory, fulfillment, notification. Each step runs independently with the ability to halt on failure.

## Deep Dive

### Simple Chain

```ruby
class ValidateOrderJob < ApplicationJob
  def perform(order_id)
    order = Order.find(order_id)
    OrderValidator.validate!(order)
    order.update!(status: :validated)
    ChargePaymentJob.perform_later(order_id)
  end
end

class ChargePaymentJob < ApplicationJob
  def perform(order_id)
    order = Order.find(order_id)
    return unless order.validated?
    PaymentService.charge(order)
    order.update!(status: :charged)
    FulfillOrderJob.perform_later(order_id)
  end
end
```

### State Machine Pattern

```ruby
class Order < ApplicationRecord
  enum :status, { pending: 0, validated: 1, charged: 2, fulfilled: 3 }

  after_commit :trigger_next_step, on: :update, if: :saved_change_to_status?

  private

  def trigger_next_step
    case status
    when "validated" then ChargePaymentJob.perform_later(id)
    when "charged"   then FulfillOrderJob.perform_later(id)
    end
  end
end
```

## Common Pitfalls
1. **Not guarding each step** — Without status checks, retries re-trigger completed steps.
2. **Tight coupling** — Use record status as the contract between steps.

## Best Practices
1. **Use status columns as coordination points.**
2. **Keep each job focused on one step.**

## Summary
- Job chaining breaks workflows into focused, retriable steps.
- State machines use model status to coordinate.
- Always guard each step by checking current status.

## Code Examples

**Starting an order workflow — each job triggers the next**

```ruby
# Start the workflow
ValidateOrderJob.perform_later(order.id)
# Chain: Validate → Charge → Fulfill → Notify
```


## Resources

- [Active Job Basics](https://guides.rubyonrails.org/active_job_basics.html) — Official Active Job guide

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*