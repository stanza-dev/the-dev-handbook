---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-testing-enqueuing"
---

# Testing Job Enqueuing

## Introduction
Rails provides dedicated test helpers to verify that jobs are enqueued at the right time with the right arguments, without actually executing them.

## Key Concepts
- **assert_enqueued_with**: Asserts a specific job was enqueued with specific arguments.
- **assert_enqueued_jobs**: Asserts a number of jobs were enqueued.
- **queue_adapter = :test**: Captures jobs without executing them.

## Real World Context
When a user submits an order, verify the job was enqueued with the correct order ID — not that the order was processed.

## Deep Dive

```ruby
class OrdersControllerTest < ActionDispatch::IntegrationTest
  test "creating order enqueues processing job" do
    assert_enqueued_with(job: ProcessOrderJob) do
      post orders_path, params: { order: { product_id: 1 } }
    end
  end

  test "enqueues with correct arguments" do
    post orders_path, params: { order: { product_id: 1 } }
    assert_enqueued_with(job: ProcessOrderJob, args: [Order.last.id])
  end
end
```

### Counting Jobs

```ruby
test "bulk import enqueues one job per batch" do
  assert_enqueued_jobs 5 do
    BulkImportCoordinatorJob.perform_now(import.id)
  end
end
```

## Common Pitfalls
1. **Testing enqueuing and execution in same test** — Keep them separate.
2. **Forgetting assert_enqueued_with scope** — Use a block to scope assertions.

## Best Practices
1. **Use assert_enqueued_with with a block** — Scope to exactly the code that should enqueue.
2. **Test arguments explicitly** — Verify correct IDs and parameters.

## Summary
- The :test adapter captures jobs without executing.
- assert_enqueued_with verifies specific jobs.
- Test enqueuing separately from execution.

## Code Examples

**Testing that creating a user enqueues the welcome email job**

```ruby
test "signup enqueues welcome email job" do
  assert_enqueued_with(job: SendWelcomeEmailJob) do
    User.create!(name: "Alice", email: "alice@example.com")
  end
end
```


## Resources

- [Testing Active Job](https://guides.rubyonrails.org/testing.html#testing-jobs) — Official Rails testing guide for jobs

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*