---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-testing-authentication"
---

# Testing Authentication

## Introduction

Most Rails applications protect certain actions behind authentication. Testing these protected endpoints requires you to simulate a logged-in user in your test suite. Whether you use the Rails 8 authentication generator, Devise, or a hand-rolled session-based system, the testing patterns are remarkably similar: create a helper that signs a user in, then call it before exercising protected routes.

## Key Concepts

- **Authenticated vs unauthenticated access**: Verifying that protected actions reject anonymous users and allow signed-in users.
- **Test sign-in helpers**: Reusable methods added to `ActionDispatch::IntegrationTest` that POST credentials to establish a session.
- **Authorization testing**: Confirming that a signed-in user can only access resources they are permitted to see or modify.
- **API token authentication**: Testing endpoints that require a `Bearer` token in the `Authorization` header instead of a cookie-based session.

## Real World Context

A broken authentication gate is one of the most dangerous bugs in a web application. If a before-action filter is accidentally removed, any user can access admin pages or modify other users' data. Controller tests for authentication act as a safety net: they fail immediately if a protected endpoint becomes publicly accessible, catching the mistake before it reaches production.

## Deep Dive

Start by testing that unauthenticated users are rejected. This ensures your before-action filters are in place:

```ruby
class ArticlesControllerTest < ActionDispatch::IntegrationTest
  test 'redirects to login when not authenticated' do
    get new_article_url
    assert_redirected_to login_url
  end

  test 'returns unauthorized for API without token' do
    get api_articles_url
    assert_response :unauthorized
  end
end
```

The first test confirms browser-facing routes redirect to a login page. The second confirms API routes return a 401 status.

Next, create a sign-in helper. Rails 8 ships with an authentication generator (`bin/rails generate authentication`) that creates a session-based auth system. You can write a helper that matches its login endpoint:

```ruby
# test/test_helper.rb
class ActionDispatch::IntegrationTest
  def sign_in(user)
    post session_url, params: {
      email: user.email,
      password: 'password'
    }
  end
end
```

This helper POSTs to the session endpoint, establishing a cookie-based session for subsequent requests in the same test. Now use it in your tests:

```ruby
class ArticlesControllerTest < ActionDispatch::IntegrationTest
  def setup
    @user = users(:david)
  end

  test 'can create article when logged in' do
    sign_in(@user)

    assert_difference('Article.count', 1) do
      post articles_url, params: {
        article: { title: 'New', body: 'Content' }
      }
    end
  end

  test 'can only edit own articles' do
    sign_in(@user)
    other_article = articles(:by_other_user)

    patch article_url(other_article), params: {
      article: { title: 'Hacked' }
    }

    assert_response :forbidden
  end
end
```

The first test confirms a signed-in user can create a resource. The second verifies that authorization logic prevents users from editing resources they do not own.

For API endpoints that use token-based authentication, pass the token in the request headers:

```ruby
class Api::ArticlesControllerTest < ActionDispatch::IntegrationTest
  def setup
    @user = users(:david)
    @token = @user.generate_api_token
  end

  test 'can access with valid token' do
    get api_articles_url, headers: {
      'Authorization' => "Bearer #{@token}"
    }
    assert_response :success
  end

  test 'returns unauthorized with expired token' do
    expired_token = @user.generate_api_token(expires_at: 1.hour.ago)
    get api_articles_url, headers: {
      'Authorization' => "Bearer #{expired_token}"
    }
    assert_response :unauthorized
  end
end
```

The `headers` hash lets you set any HTTP header. This pattern works for Bearer tokens, API keys, or custom authentication schemes.

## Common Pitfalls

1. **Hardcoding passwords in fixtures** — Store a consistent password hash in your fixture YAML and reference a known plaintext value in your helper. If the fixture password and the helper value drift apart, every auth test will fail with a cryptic redirect.
2. **Forgetting to test the unauthenticated path** — It is tempting to only test the happy path where the user is signed in. Always add at least one test per controller that confirms anonymous requests are rejected.
3. **Sharing sessions across tests** — Each test method starts with a clean session. Do not assume a `sign_in` from one test carries over to another.

## Best Practices

1. **Extract sign-in to a shared helper** — Define it once in `test_helper.rb` so every test file uses the same mechanism. When the auth system changes, you update one place.
2. **Test role-based access** — If your app has admin and regular user roles, write tests for both to confirm that non-admin users cannot access admin endpoints.
3. **Use the Rails 8 authentication generator** — It gives you a tested, secure baseline. Write your controller tests against its session endpoint rather than reinventing the sign-in flow.

## Summary

- Always test that unauthenticated requests are rejected with a redirect or 401 status.
- Create a shared `sign_in` helper in `test_helper.rb` that POSTs credentials to your login endpoint.
- Test authorization by attempting actions as users with different ownership or role levels.
- For APIs, pass tokens via the `Authorization` header and test both valid and invalid tokens.
- Rails 8's authentication generator provides a solid foundation for session-based auth testing.

## Code Examples

**A sign-in helper and controller tests that verify admin-only access, role-based authorization, and anonymous rejection**

```ruby
# test/test_helper.rb — shared sign-in helper
class ActionDispatch::IntegrationTest
  # Works with the Rails 8 authentication generator
  def sign_in(user)
    post session_url, params: {
      email: user.email,
      password: 'password'
    }
  end
end

# test/controllers/admin/dashboard_controller_test.rb
class Admin::DashboardControllerTest < ActionDispatch::IntegrationTest
  test 'admin can access dashboard' do
    sign_in(users(:admin))
    get admin_dashboard_url
    assert_response :success
  end

  test 'regular user is forbidden' do
    sign_in(users(:regular))
    get admin_dashboard_url
    assert_response :forbidden
  end

  test 'anonymous user is redirected to login' do
    get admin_dashboard_url
    assert_redirected_to login_url
  end
end
```


## Resources

- [Rails Testing Guide — Integration Testing](https://guides.rubyonrails.org/testing.html#integration-testing) — Official Rails guide section on integration-style testing used for controller tests
- [Rails 8 Authentication Generator](https://guides.rubyonrails.org/getting_started.html) — Overview of the built-in authentication generator shipped with Rails 8

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*