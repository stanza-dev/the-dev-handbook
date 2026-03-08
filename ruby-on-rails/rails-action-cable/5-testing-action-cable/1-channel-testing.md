---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-channel-testing"
---

# Testing Channels

## Introduction
Rails provides dedicated test helpers for Action Cable that let you verify channel behavior without establishing real WebSocket connections. Testing channels is as straightforward as testing controllers — you subscribe, perform actions, and assert on the results.

## Key Concepts
- **`ActionCable::Channel::TestCase`**: The base test class for channel tests. It provides helpers to subscribe, perform actions, and assert on streams and transmissions.
- **`subscribe` Helper**: Simulates a client subscribing to the channel with optional parameters.
- **`assert_has_stream`**: Verifies that the subscription is streaming from a specific stream name.
- **`assert_has_stream_for`**: Verifies streaming from a model-derived stream name (matches `stream_for`).
- **`perform` Helper**: Simulates a client calling a channel action with data.

## Real World Context
Without channel tests, you cannot verify that your authorization logic works, that streams are set up correctly, or that channel actions process data as expected. Channel tests catch bugs like missing `stream_for` calls, broken authorization, and incorrect broadcast payloads before they reach production.

## Deep Dive
Channel tests live in `test/channels/` and inherit from `ActionCable::Channel::TestCase`:

```ruby
# test/channels/chat_channel_test.rb
require "test_helper"

class ChatChannelTest < ActionCable::Channel::TestCase
  test "subscribes to the room stream" do
    room = rooms(:general)

    # Stub the connection's current_user
    stub_connection current_user: users(:alice)

    # Subscribe with parameters
    subscribe(room_id: room.id)

    # Assert the subscription was successful
    assert subscription.confirmed?

    # Assert streaming from the correct stream
    assert_has_stream_for room
  end

  test "rejects subscription when user is not a room member" do
    room = rooms(:private_room)
    non_member = users(:bob)

    stub_connection current_user: non_member
    subscribe(room_id: room.id)

    assert subscription.rejected?
  end

  test "speak action broadcasts the message" do
    room = rooms(:general)
    stub_connection current_user: users(:alice)
    subscribe(room_id: room.id)

    # Perform the channel action
    perform :speak, body: "Hello, world!"

    # Assert a message was created
    assert_equal "Hello, world!", Message.last.body
  end
end
```

The `stub_connection` method sets up the connection identifiers (like `current_user`) without needing a real WebSocket connection.

To test that broadcasts were sent, use `assert_broadcast_on`:

```ruby
test "speak action broadcasts to the room" do
  room = rooms(:general)
  stub_connection current_user: users(:alice)
  subscribe(room_id: room.id)

  assert_broadcast_on(ChatChannel.broadcasting_for(room)) do
    perform :speak, body: "Hello!"
  end
end
```

The `assert_broadcast_on` block captures any broadcasts sent during its execution and verifies they went to the correct stream.

For RSpec users, the equivalent setup uses `have_stream_for` matchers:

```ruby
# spec/channels/chat_channel_spec.rb
RSpec.describe ChatChannel, type: :channel do
  before { stub_connection current_user: create(:user) }

  it "subscribes to the room stream" do
    room = create(:room)
    subscribe(room_id: room.id)
    expect(subscription).to be_confirmed
    expect(subscription).to have_stream_for(room)
  end
end
```

## Common Pitfalls
1. **Forgetting `stub_connection`** — Without stubbing the connection, `current_user` is nil, and your authorization checks will behave unexpectedly.
2. **Testing broadcasts without `assert_broadcast_on`** — Simply calling `perform` and checking the database is not enough. You must verify that the correct data was broadcast to the correct stream.
3. **Not testing the rejection path** — Always test that unauthorized users are properly rejected, not just that authorized users can subscribe.

## Best Practices
1. **Test both acceptance and rejection** — Every channel test file should cover authorized subscriptions, unauthorized rejections, and edge cases like missing parameters.
2. **Use fixtures or factories for test data** — Set up rooms, users, and memberships in fixtures so your tests are clear and repeatable.
3. **Test channel actions independently** — Each action (speak, typing, etc.) should have its own test verifying both the side effects and the broadcast.

## Summary
- Channel tests inherit from `ActionCable::Channel::TestCase` and simulate subscriptions without real WebSockets.
- Use `stub_connection` to set up connection identifiers like `current_user`.
- `assert_has_stream_for` verifies correct stream setup; `assert_broadcast_on` verifies correct broadcasting.
- Always test both authorized and rejected subscription paths.
- Test channel actions independently, verifying both side effects and broadcasts.

## Code Examples

**Channel test covering both subscription acceptance and rejection — stub_connection provides the current_user without a real WebSocket**

```ruby
class ChatChannelTest < ActionCable::Channel::TestCase
  test "subscribes to the room stream" do
    room = rooms(:general)
    stub_connection current_user: users(:alice)
    subscribe(room_id: room.id)

    assert subscription.confirmed?
    assert_has_stream_for room
  end

  test "rejects non-members" do
    stub_connection current_user: users(:bob)
    subscribe(room_id: rooms(:private_room).id)

    assert subscription.rejected?
  end
end
```


## Resources

- [Testing Action Cable](https://guides.rubyonrails.org/testing.html#testing-action-cable) — Official Rails testing guide section on Action Cable tests

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*