---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-testing-email-workflows"
---

# Testing Email Workflows

## Introduction

Email is a critical communication channel in most Rails applications. Welcome emails, password resets, order confirmations, and notification digests all depend on Action Mailer working correctly. Testing email workflows ensures that the right emails are sent to the right people at the right time, with the right content. Rails provides excellent testing support through `assert_emails`, `assert_enqueued_emails`, and direct access to delivered messages.

## Key Concepts

- **ActionMailer::Base.deliveries**: An array that collects all emails sent during a test (when using the `:test` delivery method). You can inspect recipients, subjects, and body content.
- **assert_emails**: A block-based assertion that verifies exactly N emails were delivered during the block's execution.
- **assert_enqueued_emails**: Similar to `assert_emails` but checks emails that were enqueued via Active Job rather than delivered synchronously.
- **assert_no_emails**: Verifies that no emails were sent during a block, useful for testing suppression conditions.
- **Mailer previews**: Rails lets you preview emails in the browser at `/rails/mailers`, but in tests you assert on the delivered mail objects directly.

## Real World Context

Imagine your application sends a welcome email when a user signs up, but a recent code change accidentally broke the mailer. Without email tests, you would not discover this until a customer complains they never received their welcome email — possibly days later. Email workflow tests catch these regressions immediately. They also serve as documentation for your email-sending logic: reading the tests tells you exactly which actions trigger which emails.

## Deep Dive

Rails configures the test environment to use the `:test` delivery method, which stores emails in `ActionMailer::Base.deliveries` instead of actually sending them. This makes it straightforward to inspect emails in your tests.

Start by testing that a mailer delivers an email with the correct attributes:

```ruby
require 'test_helper'

class UserMailerTest < ActionMailer::TestCase
  test 'welcome email has correct recipient and subject' do
    user = users(:david)
    email = UserMailer.welcome(user)

    assert_emails 1 do
      email.deliver_now
    end

    assert_equal ['david@example.com'], email.to
    assert_equal 'Welcome to our platform!', email.subject
    assert_match 'Hi David', email.body.encoded
  end
end
```

The `email.to` field returns an array of recipient addresses. The `email.body.encoded` method returns the full email body as a string, which you can search with `assert_match`.

For integration tests that trigger emails as a side effect of a controller action, use `assert_emails` to verify the email count:

```ruby
class RegistrationFlowTest < ActionDispatch::IntegrationTest
  test 'signing up sends a welcome email' do
    assert_emails 1 do
      post users_url, params: {
        user: {
          name: 'Alice',
          email: 'alice@example.com',
          password: 'secure123',
          password_confirmation: 'secure123'
        }
      }
    end

    # Inspect the delivered email
    welcome_email = ActionMailer::Base.deliveries.last
    assert_equal ['alice@example.com'], welcome_email.to
    assert_equal 'Welcome to our platform!', welcome_email.subject
  end
end
```

If your mailers use Active Job for asynchronous delivery (via `deliver_later`), use `assert_enqueued_emails` instead:

```ruby
test 'order confirmation is enqueued for async delivery' do
  sign_in(users(:buyer))

  assert_enqueued_emails 1 do
    post orders_url, params: {
      order: { shipping_address: '123 Main St' }
    }
  end
end
```

You can also test that certain conditions suppress email delivery. For example, a user who opted out of marketing emails should not receive a newsletter:

```ruby
test 'does not send newsletter to opted-out users' do
  user = users(:opted_out)

  assert_no_emails do
    NewsletterMailer.weekly_digest(user).deliver_now
  end
end
```

For multipart emails (HTML + plain text), test both parts independently:

```ruby
test 'welcome email has HTML and text parts' do
  user = users(:david)
  email = UserMailer.welcome(user)

  assert_equal 2, email.parts.size

  html_part = email.html_part.body.encoded
  text_part = email.text_part.body.encoded

  assert_match '<h1>Welcome</h1>', html_part
  assert_match 'Welcome', text_part
  # Both parts should contain the user's name
  assert_match user.name, html_part
  assert_match user.name, text_part
end
```

Testing both parts ensures that users who view emails in plain-text clients still see meaningful content.

## Common Pitfalls

1. **Not clearing the deliveries array** — `ActionMailer::Base.deliveries` persists across tests unless cleared. Add `setup { ActionMailer::Base.deliveries.clear }` to test classes that inspect specific emails, or rely on `assert_emails` blocks which scope the count automatically.
2. **Testing deliver_later with assert_emails** — `assert_emails` checks synchronous delivery. If your mailer uses `deliver_later`, the email is enqueued in Active Job instead. Use `assert_enqueued_emails` or set `config.active_job.queue_adapter = :inline` in your test environment to force synchronous execution.
3. **Asserting on the full email body** — Email bodies contain HTML tags, encoding artifacts, and whitespace. Use `assert_match` with a short, stable substring rather than comparing the entire body string.

## Best Practices

1. **Test mailers at two levels** — Write unit tests in `test/mailers/` for email content and formatting. Write integration tests that verify emails are triggered by the correct controller actions.
2. **Use fixtures for email test data** — Reference fixture users and orders in your mailer tests so the test data is consistent and easy to update.
3. **Verify critical email fields** — Always assert on `to`, `subject`, and at least one key phrase in the body. These are the fields most likely to break during refactoring.

## Summary

- Rails stores test emails in `ActionMailer::Base.deliveries` for inspection, with no emails actually sent.
- Use `assert_emails N` for synchronous delivery and `assert_enqueued_emails N` for jobs queued via `deliver_later`.
- Inspect individual emails by accessing `ActionMailer::Base.deliveries.last` and asserting on `to`, `subject`, and `body.encoded`.
- Use `assert_no_emails` to verify that emails are correctly suppressed under certain conditions.
- Test both HTML and plain-text parts of multipart emails to ensure all clients receive readable content.

## Code Examples

**Mailer tests verifying email delivery with content assertions and testing email suppression for opted-out users**

```ruby
class NotificationMailerTest < ActionMailer::TestCase
  setup do
    ActionMailer::Base.deliveries.clear
  end

  test 'sends order shipped notification' do
    order = orders(:recent)

    assert_emails 1 do
      NotificationMailer.order_shipped(order).deliver_now
    end

    email = ActionMailer::Base.deliveries.last
    assert_equal [order.user.email], email.to
    assert_equal "Your order ##{order.number} has shipped!", email.subject
    assert_match order.tracking_number, email.body.encoded
  end

  test 'does not notify if user disabled email notifications' do
    order = orders(:by_silent_user)

    assert_no_emails do
      NotificationMailer.order_shipped(order).deliver_now
    end
  end
end
```


## Resources

- [Rails Testing Guide — Testing Your Mailers](https://guides.rubyonrails.org/testing.html#testing-your-mailers) — Official Rails guide on unit and integration testing for Action Mailer classes
- [Action Mailer Basics](https://guides.rubyonrails.org/action_mailer_basics.html) — Complete guide to creating and configuring mailers in Rails

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*