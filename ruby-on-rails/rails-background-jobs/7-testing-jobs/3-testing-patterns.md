---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-testing-patterns"
---

# Advanced Testing Patterns

## Introduction
Real-world job testing requires handling time, stubbing external services, and verifying negative cases.

## Key Concepts
- **Time Helpers**: `travel_to`, `freeze_time` for time-dependent tests.
- **Stubbing**: Replacing external calls with controlled responses.

## Real World Context
A job calling Stripe needs stubbed API responses. A daily digest needs time-travel tests.

## Deep Dive

### Stubbing External Services

```ruby
test "syncs customer to Stripe" do
  Stripe::Customer.stub(:create, OpenStruct.new(id: "cus_123")) do
    SyncToStripeJob.perform_now(user.id)
  end
  assert_equal "cus_123", user.reload.stripe_customer_id
end
```

### Negative Tests

```ruby
test "no job for deactivated users" do
  assert_no_enqueued_jobs do
    OnboardingJob.perform_now(deactivated_user.id)
  end
end
```

### Queue Assertions

```ruby
test "enqueues to critical queue" do
  assert_enqueued_with(job: PaymentJob, queue: "critical") do
    PaymentService.charge(order)
  end
end
```

## Common Pitfalls
1. **Testing against real external services** — Slow, flaky, costly.
2. **Not testing discard paths** — Verify discard_on side effects.

## Best Practices
1. **Stub all external services.**
2. **Test every retry_on and discard_on path.**

## Summary
- Stub external services for reliable tests.
- Write negative tests for skip conditions.
- Cover every error handling path.

## Code Examples

**freeze_time for testing delayed scheduling**

```ruby
test "schedules follow-up 3 days later" do
  freeze_time do
    assert_enqueued_with(job: FollowUpJob, at: 3.days.from_now) do
      OnboardingJob.perform_now(user.id)
    end
  end
end
```


## Resources

- [Rails Testing Guide](https://guides.rubyonrails.org/testing.html) — Comprehensive testing guide

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*