---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-testing-connections"
---

# Testing Connections

## Introduction
Connection tests verify that your WebSocket authentication works correctly — that valid users connect successfully and invalid users are rejected. Since the Connection class is the gateway to all of Action Cable, testing it thoroughly prevents unauthorized access to every channel in your application.

## Key Concepts
- **`ActionCable::Connection::TestCase`**: The base test class for connection tests. It provides helpers to simulate WebSocket connections with cookies, headers, and parameters.
- **`connect` Helper**: Simulates a WebSocket connection attempt with configurable cookies, headers, and params.
- **`assert_reject_connection`**: Asserts that the connection was rejected (i.e., `reject_unauthorized_connection` was called).
- **`cookies` Hash**: Allows setting cookies that the Connection's `connect` method can read for authentication.

## Real World Context
If your Connection class reads `cookies.encrypted[:user_id]` to authenticate, you need to verify that: (1) a valid cookie results in a successful connection with the correct `current_user`, (2) an invalid or missing cookie results in rejection, and (3) a cookie for a deleted user results in rejection. These are the same scenarios you would test for HTTP authentication, applied to WebSockets.

## Deep Dive
Connection tests live in `test/channels/application_cable/` or `test/channels/`:

```ruby
# test/channels/application_cable/connection_test.rb
require "test_helper"

class ApplicationCable::ConnectionTest < ActionCable::Connection::TestCase
  test "connects with a valid user cookie" do
    user = users(:alice)

    # Set encrypted cookie matching what the app uses
    cookies.encrypted[:user_id] = user.id

    # Simulate the WebSocket connection
    connect

    # Verify the connection identifier was set
    assert_equal user, connection.current_user
  end

  test "rejects connection without a cookie" do
    assert_reject_connection { connect }
  end

  test "rejects connection with an invalid user id" do
    cookies.encrypted[:user_id] = -1

    assert_reject_connection { connect }
  end

  test "rejects connection for a deleted user" do
    cookies.encrypted[:user_id] = 999_999  # Non-existent user

    assert_reject_connection { connect }
  end
end
```

The `connect` method simulates the WebSocket upgrade handshake. You can also pass headers and parameters:

```ruby
test "connects with a token in params" do
  user = users(:alice)
  token = user.generate_websocket_token

  connect params: { token: token }

  assert_equal user, connection.current_user
end

test "connects with a token in headers" do
  user = users(:alice)
  token = user.generate_websocket_token

  connect headers: { "Authorization" => "Bearer #{token}" }

  assert_equal user, connection.current_user
end
```

For testing disconnection behavior:

```ruby
test "disconnect cleans up user presence" do
  user = users(:alice)
  cookies.encrypted[:user_id] = user.id

  connect
  assert_equal user, connection.current_user

  disconnect

  # Verify cleanup happened
  user.reload
  assert_nil user.last_seen_at
end
```

## Common Pitfalls
1. **Not testing the rejection path** — Most developers test that valid users connect, but forget to test that invalid users are rejected. Both paths are equally important.
2. **Mismatching cookie encryption** — If your Connection reads `cookies.encrypted[:user_id]`, your test must set `cookies.encrypted[:user_id]`, not `cookies[:user_id]`. The encryption method must match.
3. **Ignoring edge cases** — Test what happens when the user exists but is suspended, banned, or has an expired session. These are common vectors for unauthorized access.

## Best Practices
1. **Test every authentication path** — If your Connection supports both cookie and token auth, test both paths, including their failure modes.
2. **Test disconnection cleanup** — If your `disconnect` method updates presence or cleans up resources, verify that behavior in tests.
3. **Keep connection tests focused on authentication** — Do not test channel behavior in connection tests. Each test class has a single responsibility.

## Summary
- Connection tests inherit from `ActionCable::Connection::TestCase` and verify WebSocket authentication.
- Use `connect` with cookies, headers, or params to simulate connection attempts.
- `assert_reject_connection` verifies that invalid users are properly denied.
- Test every authentication path and edge case — invalid cookies, missing tokens, deleted users.
- Test disconnection cleanup if your `disconnect` method performs side effects.

## Code Examples

**Connection tests — verify both successful authentication and rejection of unauthorized connection attempts**

```ruby
class ApplicationCable::ConnectionTest < ActionCable::Connection::TestCase
  test "connects with a valid cookie" do
    user = users(:alice)
    cookies.encrypted[:user_id] = user.id
    connect
    assert_equal user, connection.current_user
  end

  test "rejects without a cookie" do
    assert_reject_connection { connect }
  end
end
```


## Resources

- [Testing Action Cable Connections](https://guides.rubyonrails.org/testing.html#connection-test-case) — Official guide on testing Action Cable connection authentication

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*