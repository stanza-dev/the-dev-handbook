---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-testing-broadcasts"
---

# Testing Broadcasts and Integration

## Introduction
Beyond unit-testing individual channels and connections, you need to verify that your application correctly broadcasts data when models change, controllers act, or jobs run. Broadcast assertions let you confirm that the right data reaches the right stream at the right time.

## Key Concepts
- **`assert_broadcasts`**: Asserts that a specific number of messages were broadcast to a stream during a block.
- **`assert_broadcast_on`**: Asserts that a specific message was broadcast to a specific stream.
- **`assert_no_broadcasts`**: Asserts that no messages were broadcast to a stream during a block.
- **Integration Testing**: Testing the full flow from an HTTP request or model change through to the WebSocket broadcast.

## Real World Context
Unit-testing a channel confirms that the subscription and actions work in isolation. But in production, broadcasts are triggered by controller actions, model callbacks, and background jobs. Integration tests verify the complete chain — from the event that triggers a broadcast to the data that reaches subscribers.

## Deep Dive
Broadcast assertions work with any test class. Here is a model test verifying that creating a record triggers a broadcast:

```ruby
# test/models/message_test.rb
require "test_helper"

class MessageTest < ActiveSupport::TestCase
  test "broadcasting a message after creation" do
    room = rooms(:general)
    user = users(:alice)

    assert_broadcasts(ChatChannel.broadcasting_for(room), 1) do
      Message.create!(room: room, user: user, body: "Hello!")
    end
  end

  test "does not broadcast on validation failure" do
    room = rooms(:general)
    user = users(:alice)

    assert_no_broadcasts(ChatChannel.broadcasting_for(room)) do
      Message.create(room: room, user: user, body: "")  # blank body fails validation
    end
  end
end
```

To assert the exact content of a broadcast:

```ruby
test "broadcasts the correct payload" do
  room = rooms(:general)
  user = users(:alice)

  assert_broadcast_on(ChatChannel.broadcasting_for(room), {
    type: "new_message",
    data: {
      body: "Hello!",
      user: user.username
    }
  }.with_indifferent_access) do
    Message.create!(room: room, user: user, body: "Hello!")
  end
end
```

For controller integration tests:

```ruby
# test/controllers/messages_controller_test.rb
class MessagesControllerTest < ActionDispatch::IntegrationTest
  test "creating a message broadcasts to the room" do
    room = rooms(:general)
    sign_in users(:alice)

    assert_broadcasts(ChatChannel.broadcasting_for(room), 1) do
      post room_messages_path(room), params: {
        message: { body: "Hello from controller test!" }
      }
    end

    assert_response :created
  end
end
```

Testing broadcasts from background jobs requires `perform_enqueued_jobs`:

```ruby
test "notification job broadcasts to the user" do
  user = users(:alice)
  notification = notifications(:new_comment)

  assert_broadcasts(NotificationsChannel.broadcasting_for(user), 1) do
    perform_enqueued_jobs do
      NotificationBroadcastJob.perform_later(notification)
    end
  end
end
```

## Common Pitfalls
1. **Forgetting `perform_enqueued_jobs` when testing job-based broadcasts** — If your broadcast is triggered by a background job, the job must actually execute during the test for the broadcast assertion to pass.
2. **Mismatching stream names in assertions** — Use `ChatChannel.broadcasting_for(room)` to generate the stream name consistently. Hand-typing stream names in assertions leads to false passes when the names do not match.
3. **Not testing the no-broadcast case** — Verifying that invalid operations do NOT trigger broadcasts is just as important as verifying valid ones do.

## Best Practices
1. **Use `broadcasting_for` to generate stream names** — This ensures your assertion matches the actual stream name used by `stream_for` and `broadcast_to`.
2. **Test both positive and negative cases** — Assert that valid actions broadcast and that invalid actions do not.
3. **Test the complete chain in integration tests** — Verify from HTTP request through controller, model, job, and broadcast.

## Summary
- `assert_broadcasts` counts messages sent to a stream during a block.
- `assert_broadcast_on` verifies the exact content of a broadcast message.
- `assert_no_broadcasts` confirms that no messages were sent, critical for validation failure paths.
- Use `broadcasting_for` to generate consistent stream names in assertions.
- Wrap job-based broadcasts in `perform_enqueued_jobs` to execute the job during the test.

## Code Examples

**Broadcast assertions — verify that model callbacks trigger the correct broadcasts and that failures do not broadcast**

```ruby
class MessageTest < ActiveSupport::TestCase
  test "broadcasts after creation" do
    room = rooms(:general)

    assert_broadcasts(ChatChannel.broadcasting_for(room), 1) do
      Message.create!(room: room, user: users(:alice), body: "Hello!")
    end
  end

  test "does not broadcast on failure" do
    room = rooms(:general)

    assert_no_broadcasts(ChatChannel.broadcasting_for(room)) do
      Message.create(room: room, user: users(:alice), body: "")
    end
  end
end
```


## Resources

- [Action Cable Testing Helpers](https://api.rubyonrails.org/classes/ActionCable/TestHelper.html) — API docs for all Action Cable test assertion methods

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*