---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-integration-test-concepts"
---

# Integration Test Concepts

## Introduction

Integration tests verify that multiple parts of your Rails application work together correctly. While model tests check individual business rules and controller tests verify single request/response cycles, integration tests exercise complete user workflows that span multiple controllers, sessions, and redirects. They answer the question: "Does the user journey actually work from start to finish?"

## Key Concepts

- **User workflow testing**: Simulating a sequence of HTTP requests that mirror what a real user would do, such as signing up, logging in, and creating content.
- **follow_redirect!**: A test helper that follows an HTTP redirect, allowing you to assert on the final destination page rather than the intermediate redirect response.
- **Session persistence**: Integration tests maintain session state across multiple requests within the same test method, just like a real browser would.
- **Cross-controller flows**: Testing interactions that span multiple controllers, such as a login on the sessions controller followed by actions on the articles controller.

## Real World Context

Consider an e-commerce checkout flow: a user logs in, adds items to a cart, proceeds to checkout, enters shipping details, and confirms the order. Each step involves a different controller and different database operations. A model test can verify that an order validates its address, and a controller test can confirm the checkout page renders, but only an integration test catches bugs like "the cart is empty after login because the session was not preserved." Integration tests give you end-to-end confidence without the slowness of a full browser test.

## Deep Dive

Integration tests inherit from `ActionDispatch::IntegrationTest`, the same base class used by controller tests. The difference is in scope: integration tests intentionally hit multiple endpoints in sequence.

Here is a test that verifies a complete signup-to-dashboard flow:

```ruby
require 'test_helper'

class UserSignupFlowTest < ActionDispatch::IntegrationTest
  test 'user can sign up and access protected content' do
    # Visit signup page
    get signup_url
    assert_response :success

    # Submit registration form
    assert_difference('User.count', 1) do
      post users_url, params: {
        user: {
          name: 'New User',
          email: 'new@example.com',
          password: 'password123',
          password_confirmation: 'password123'
        }
      }
    end

    # Should be redirected to dashboard
    follow_redirect!
    assert_response :success
    assert_match 'Welcome', response.body

    # Can access protected page
    get dashboard_url
    assert_response :success
  end
end
```

The `follow_redirect!` call is essential. Without it, the test would stop at the 302 response and never verify that the dashboard actually renders. This helper follows the redirect and makes the subsequent assertions against the destination page.

Session state persists across requests within a single test method. This means you can log in once and make authenticated requests for the rest of the test:

```ruby
test 'session persists across requests' do
  post login_url, params: {
    email: 'user@example.com',
    password: 'password'
  }

  # Session maintained in subsequent requests
  get profile_url
  assert_response :success

  get settings_url
  assert_response :success

  # Until logout
  delete logout_url

  get profile_url
  assert_redirected_to login_url
end
```

After the `delete logout_url` request destroys the session, subsequent requests are treated as anonymous. This is exactly how a browser behaves when a user clicks "Log out."

You can also test redirect chains, where one action redirects to another which may redirect again:

```ruby
test 'login redirects to intended destination' do
  # Try to access protected page
  get admin_dashboard_url
  assert_redirected_to login_url

  # Login
  post login_url, params: {
    email: 'admin@example.com',
    password: 'password'
  }

  # Follow redirect back to admin dashboard
  follow_redirect!
  assert_equal admin_dashboard_path, path
end
```

The `path` helper returns the path of the current page after following redirects, which is useful for verifying that the user ended up in the right place.

## Common Pitfalls

1. **Forgetting follow_redirect!** — If your action redirects, calling `assert_response :success` immediately will fail because the response is a 302. Use `follow_redirect!` first, then assert on the destination.
2. **Writing overly long integration tests** — A single test that covers 20 steps is hard to debug when it fails. Break long workflows into focused tests that each cover a specific user journey.
3. **Assuming session state carries between test methods** — Each test method starts fresh. A `sign_in` in one test does not carry over to the next.

## Best Practices

1. **Name tests after user stories** — Use descriptive names like `'user can sign up and access protected content'` rather than `'test_signup'`. This makes test output readable as a specification.
2. **Test the critical path first** — Focus integration tests on your application's most important workflows: signup, login, checkout, password reset. These are the flows where bugs cause the most user pain.
3. **Combine with assert_emails for email workflows** — When a workflow sends an email (e.g., password reset), wrap the request in `assert_emails 1` to verify the email was enqueued.

## Summary

- Integration tests verify complete user workflows that span multiple controllers and requests.
- Use `follow_redirect!` to follow HTTP redirects and assert on the final destination.
- Session state persists across requests within a single test method, just like a real browser.
- Keep integration tests focused on critical user journeys rather than testing every possible path.
- Name tests descriptively to serve as living documentation of your application's behavior.

## Code Examples

**An integration test covering a multi-step checkout workflow with session persistence, redirects, and database assertions**

```ruby
class OrderCheckoutFlowTest < ActionDispatch::IntegrationTest
  test 'user completes checkout workflow' do
    sign_in(users(:buyer))

    # Add item to cart
    post cart_items_url, params: { product_id: products(:widget).id }
    follow_redirect!
    assert_match 'added to cart', response.body

    # View cart
    get cart_url
    assert_response :success

    # Submit order
    assert_difference('Order.count', 1) do
      post orders_url, params: {
        order: { shipping_address: '123 Main St' }
      }
    end

    # Verify confirmation page
    follow_redirect!
    assert_match 'Order confirmed', response.body
  end
end
```


## Resources

- [Rails Testing Guide — Integration Testing](https://guides.rubyonrails.org/testing.html#integration-testing) — Official Rails guide on writing integration tests with session management and redirect following

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*