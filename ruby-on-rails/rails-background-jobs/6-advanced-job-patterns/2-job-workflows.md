---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-job-workflows"
---

# Job Workflows and Pipelines

## Introduction
Complex business processes require orchestrating multiple jobs with dependencies, parallel execution, and compensation logic.

## Key Concepts
- **Pipeline**: A sequence of jobs where each step's output feeds the next.
- **Saga Pattern**: Compensating transactions that undo previous steps if a later step fails.

## Real World Context
Order fulfillment: validate → charge → reserve inventory → ship → notify. If shipping fails, you need to release inventory and refund payment.

## Deep Dive

### Pipeline with Compensation

```ruby
class FulfillOrderJob < ApplicationJob
  def perform(order_id)
    order = Order.find(order_id)
    return unless order.charged?

    begin
      WarehouseService.reserve(order)
      order.update!(status: :reserved)
    rescue WarehouseService::OutOfStockError
      # Compensate: refund the charge
      PaymentService.refund(order)
      order.update!(status: :refund_pending)
      CustomerMailer.out_of_stock(order).deliver_later
    end
  end
end
```

### Parallel Fan-Out with Callback

```ruby
class DeployJob < ApplicationJob
  def perform(deploy_id)
    deploy = Deploy.find(deploy_id)

    # Fan out parallel tasks
    RunTestsJob.perform_later(deploy_id)
    BuildAssetsJob.perform_later(deploy_id)
    LintCodeJob.perform_later(deploy_id)

    # Each sub-job calls DeployCheckJob when done
  end
end

class DeployCheckJob < ApplicationJob
  def perform(deploy_id)
    deploy = Deploy.find(deploy_id)
    return unless deploy.all_checks_passed?

    PromoteToProductionJob.perform_later(deploy_id)
  end
end
```

## Common Pitfalls
1. **No compensation for failed steps** — If step 3 fails, steps 1 and 2 may need to be undone.
2. **Tight coupling between pipeline steps** — Use database status as the contract.

## Best Practices
1. **Design compensation for every step that has side effects.**
2. **Use database status columns to coordinate pipeline state.**

## Summary
- Pipelines chain jobs with dependency ordering.
- Saga patterns add compensation for failed steps.
- Fan-out patterns enable parallel execution.
- Use status columns for coordination between steps.

## Code Examples

**Saga pattern — compensates the charge if inventory reservation fails**

```ruby
# Saga: compensate on failure
class ChargeAndReserveJob < ApplicationJob
  def perform(order_id)
    order = Order.find(order_id)
    PaymentService.charge(order)
    order.update!(status: :charged)

    WarehouseService.reserve(order)
    order.update!(status: :reserved)
  rescue WarehouseService::Error
    PaymentService.refund(order)  # Compensate
    order.update!(status: :charge_refunded)
  end
end
```


## Resources

- [Active Job Basics](https://guides.rubyonrails.org/active_job_basics.html) — Official Active Job guide

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*