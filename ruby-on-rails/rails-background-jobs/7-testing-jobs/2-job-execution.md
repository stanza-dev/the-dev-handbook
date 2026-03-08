---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-testing-job-execution"
---

# Testing Job Execution

## Introduction
Beyond enqueuing, test that jobs produce correct results using `perform_now` for unit tests and `perform_enqueued_jobs` for integration tests.

## Key Concepts
- **perform_now**: Synchronous execution for unit testing.
- **perform_enqueued_jobs**: Drains the test queue and executes all pending jobs.

## Real World Context
A payment job should create a charge, update order status, and send confirmation. Test all three outcomes.

## Deep Dive

### Unit Testing

```ruby
class ProcessOrderJobTest < ActiveJob::TestCase
  test "processes order" do
    order = orders(:pending)
    ProcessOrderJob.perform_now(order.id)
    assert_equal "processed", order.reload.status
  end

  test "handles missing records" do
    ProcessOrderJob.perform_now(-1)  # Should not raise
  end

  test "idempotent" do
    order = orders(:processed)
    ProcessOrderJob.perform_now(order.id)  # No-op
    assert_equal "processed", order.reload.status
  end
end
```

### Integration Testing

```ruby
test "full workflow" do
  perform_enqueued_jobs do
    ValidateOrderJob.perform_later(order.id)
  end
  assert_equal "fulfilled", order.reload.status
end
```

## Common Pitfalls
1. **Not testing unhappy path** — Test missing records and already-processed states.
2. **Overusing perform_enqueued_jobs** — It runs ALL pending jobs. Use perform_now for focused tests.

## Best Practices
1. **Test each job in isolation with perform_now.**
2. **Test idempotency — run the job twice and verify correctness.**

## Summary
- Use perform_now for focused unit tests.
- Use perform_enqueued_jobs for integration tests.
- Test happy paths, errors, and idempotency.

## Code Examples

**Testing idempotency — running twice should be a safe no-op**

```ruby
class ChargeJobTest < ActiveJob::TestCase
  test "idempotent — no double charge" do
    order = orders(:charged)
    ChargePaymentJob.perform_now(order.id)
    assert_equal "charged", order.reload.status  # Unchanged
  end
end
```


## Resources

- [Testing Jobs](https://guides.rubyonrails.org/testing.html#testing-jobs) — Official testing guide

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*