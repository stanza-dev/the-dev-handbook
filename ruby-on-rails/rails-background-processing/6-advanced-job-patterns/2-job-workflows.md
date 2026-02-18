---
source_course: "rails-background-processing"
source_lesson: "rails-background-jobs-job-workflows"
---

# Job Workflows

Orchestrate multiple jobs that depend on each other.

## Sequential Job Chains

```ruby
class OrderFulfillmentWorkflow
  def self.start(order_id)
    # Jobs run in sequence
    ValidateOrderJob.perform_later(order_id)
  end
end

class ValidateOrderJob < ApplicationJob
  def perform(order_id)
    order = Order.find(order_id)
    order.validate!
    
    # Trigger next step
    ChargePaymentJob.perform_later(order_id)
  end
end

class ChargePaymentJob < ApplicationJob
  def perform(order_id)
    order = Order.find(order_id)
    PaymentService.charge(order)
    
    # Trigger next step
    FulfillOrderJob.perform_later(order_id)
  end
end
```

## State Machine Pattern

```ruby
class Order < ApplicationRecord
  include AASM
  
  aasm column: :status do
    state :pending, initial: true
    state :validated, :charged, :fulfilled, :shipped
    
    event :validate do
      transitions from: :pending, to: :validated
      after { ChargePaymentJob.perform_later(id) }
    end
    
    event :charge do
      transitions from: :validated, to: :charged
      after { FulfillOrderJob.perform_later(id) }
    end
    
    event :fulfill do
      transitions from: :charged, to: :fulfilled
      after { ShipOrderJob.perform_later(id) }
    end
  end
end
```

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-processing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*