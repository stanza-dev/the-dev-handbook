---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-complex-workflows"
---

# Testing Complex Workflows

## Introduction

Real applications have workflows that touch many parts of the system: a checkout updates inventory, charges a payment method, sends a confirmation email, and clears the cart. Testing these workflows end-to-end ensures that all the pieces connect correctly. This lesson covers patterns for testing multi-model workflows, email-triggered flows, and processes with side effects.

## Key Concepts

- **Multi-model workflows**: Business processes that create or update records across several ActiveRecord models in a single user action.
- **assert_emails**: A test helper that verifies the exact number of emails enqueued or delivered during a block of code.
- **assert_difference with multiple expressions**: Passing multiple count expressions to `assert_difference` to verify several models changed simultaneously.
- **Token extraction from emails**: Parsing email bodies in tests to extract tokens or links needed for subsequent workflow steps like password resets.

## Real World Context

Consider a SaaS application where a user upgrades their subscription plan. This single action involves creating a new subscription record, scheduling the old plan's cancellation, prorating charges, updating the user's feature flags, and sending a confirmation email. If any of these steps fails silently, the user ends up in an inconsistent state — paying for a plan they cannot access, or accessing features they have not paid for. Complex workflow tests catch these integration failures.

## Deep Dive

A complete e-commerce checkout test exercises the full purchase flow, verifying database changes, redirects, and page content at each step:

```ruby
class CheckoutFlowTest < ActionDispatch::IntegrationTest
  def setup
    @user = users(:buyer)
    @product = products(:widget)
  end

  test 'complete checkout process' do
    post login_url, params: {
      email: @user.email,
      password: 'password'
    }

    # Add item to cart
    post cart_items_url, params: {
      product_id: @product.id,
      quantity: 2
    }
    assert_response :redirect
    follow_redirect!

    # View cart
    get cart_url
    assert_response :success
    assert_match @product.name, response.body

    # Proceed to checkout
    get checkout_url
    assert_response :success

    # Submit order
    assert_difference(['Order.count', 'OrderItem.count'], 1) do
      post orders_url, params: {
        order: {
          shipping_address: '123 Main St',
          payment_method: 'credit_card'
        }
      }
    end

    # Verify order confirmation
    follow_redirect!
    assert_match 'Order confirmed', response.body
    assert_match Order.last.confirmation_number, response.body

    # Cart should be empty
    get cart_url
    assert_match 'Your cart is empty', response.body
  end
end
```

The `assert_difference` call with an array of expressions verifies that both `Order.count` and `OrderItem.count` increased by 1. This is a powerful technique for confirming that a single action has the expected side effects across multiple tables.

Workflows involving email are common in authentication and notification flows. Use `assert_emails` to verify emails are sent, then extract tokens from the email body to continue the workflow:

```ruby
class PasswordResetFlowTest < ActionDispatch::IntegrationTest
  test 'complete password reset flow' do
    user = users(:david)

    # Request password reset
    assert_emails 1 do
      post password_resets_url, params: {
        email: user.email
      }
    end

    # Get reset token from email
    email = ActionMailer::Base.deliveries.last
    reset_token = extract_token_from_email(email)

    # Visit reset page
    get edit_password_reset_url(reset_token)
    assert_response :success

    # Submit new password
    patch password_reset_url(reset_token), params: {
      user: {
        password: 'newpassword123',
        password_confirmation: 'newpassword123'
      }
    }

    # Should be logged in and redirected
    follow_redirect!
    assert_match 'Password has been reset', response.body

    # Can login with new password
    delete logout_url
    post login_url, params: {
      email: user.email,
      password: 'newpassword123'
    }
    assert_redirected_to dashboard_url
  end

  private

  def extract_token_from_email(email)
    email.body.encoded.match(/reset_password\/(.+?)\"/)&.[](1)
  end
end
```

The `extract_token_from_email` helper uses a regular expression to pull the reset token from the email's HTML body. This technique lets you test the complete flow without relying on external email services.

The `assert_emails` block verifies that exactly one email was enqueued during the password reset request. If the mailer is broken or the email condition is wrong, this assertion fails immediately.

## Common Pitfalls

1. **Not clearing deliveries between tests** — `ActionMailer::Base.deliveries` accumulates across tests unless cleared. Add `ActionMailer::Base.deliveries.clear` to your `setup` method or use `assert_emails` which handles scoping automatically.
2. **Brittle token extraction** — If the email template changes, your regex breaks. Keep the regex simple and add a comment explaining what it matches. Consider adding a data attribute to the email link for easier extraction.
3. **Testing too many things in one test** — A 50-line integration test that fails on line 40 is hard to debug. Split workflows into logical phases: one test for the happy path, separate tests for each failure mode.

## Best Practices

1. **Use assert_difference with arrays for multi-model changes** — `assert_difference(['Order.count', 'Payment.count'], 1)` is more expressive and fails faster than two separate assertions.
2. **Test the complete round-trip** — For password resets, test the full flow from requesting the email to logging in with the new password. Partial tests miss integration bugs.
3. **Extract helper methods for repeated sequences** — If multiple tests need to log in and add items to a cart, extract those steps into private helper methods within the test class.

## Summary

- Complex workflow tests verify that multi-step business processes work end-to-end across models and controllers.
- Use `assert_difference` with arrays to verify multiple database changes from a single action.
- Use `assert_emails` to confirm emails are sent during workflows like password resets or order confirmations.
- Extract tokens from test emails using regex to continue multi-step flows in the same test.
- Split long workflows into focused tests to make failures easier to diagnose.

## Code Examples

**An invitation workflow test spanning admin actions, email delivery, token extraction, and new user registration**

```ruby
class InvitationFlowTest < ActionDispatch::IntegrationTest
  test 'admin invites user who signs up via email link' do
    sign_in(users(:admin))

    # Admin sends invitation
    assert_emails 1 do
      assert_difference('Invitation.count', 1) do
        post invitations_url, params: {
          invitation: { email: 'new@example.com', role: 'editor' }
        }
      end
    end

    # Extract invitation token from email
    email = ActionMailer::Base.deliveries.last
    token = email.body.encoded.match(/accept\/(.+?)\"/)&.[](1)

    # New user accepts invitation
    delete logout_url
    get accept_invitation_url(token)
    assert_response :success

    # Complete registration
    assert_difference('User.count', 1) do
      post accept_invitation_url(token), params: {
        user: { name: 'New Editor', password: 'secure123' }
      }
    end

    follow_redirect!
    assert_match 'Welcome', response.body
  end
end
```


## Resources

- [Rails Testing Guide — Testing Emails](https://guides.rubyonrails.org/testing.html#testing-your-mailers) — Official Rails guide on testing mailer classes and email delivery in integration tests

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*