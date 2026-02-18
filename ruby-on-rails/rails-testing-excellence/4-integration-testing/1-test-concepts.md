---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-integration-test-concepts"
---

# Integration Test Concepts

Integration tests verify that multiple parts of your application work together correctly. They test complete user workflows.

## When to Use Integration Tests

- Testing user journeys (signup → login → use feature)
- Verifying workflows across multiple controllers
- Testing that redirects and sessions work correctly
- Validating complex business processes

## Basic Integration Test

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

## Following Redirects

Use `follow_redirect!` to follow HTTP redirects:

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

## Session Persistence

Integration tests maintain session state:

```ruby
test 'session persists across requests' do
  # Login
  post login_url, params: { email: 'user@example.com', password: 'pass' }
  
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

Learn more at [Integration Testing](https://guides.rubyonrails.org/testing.html#integration-testing).

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*