---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-stream-authorization"
---

# Stream Authorization and Rejection

## Introduction
Not every user should be able to subscribe to every channel. Action Cable provides mechanisms to authorize subscriptions and reject unauthorized access, ensuring that sensitive real-time data reaches only the users who should see it.

## Key Concepts
- **Subscription Rejection**: Calling `reject` in the `subscribed` callback refuses the subscription and notifies the client that access was denied.
- **Authorization Guard**: A check in `subscribed` that verifies the current user has permission to access the requested stream.
- **`reject` Method**: Prevents the subscription from being established. The client's `rejected` callback fires instead of `connected`.

## Real World Context
Consider a project management app with private boards. When a user tries to subscribe to the BoardChannel for a specific board, you must verify they are a member of that board. Without this check, any authenticated user could subscribe to any board's stream and see private updates. This is the WebSocket equivalent of controller authorization — and it is just as critical.

## Deep Dive
Here is a channel with proper authorization:

```ruby
# app/channels/board_channel.rb
class BoardChannel < ApplicationCable::Channel
  def subscribed
    @board = Board.find_by(id: params[:board_id])

    if @board && current_user.member_of?(@board)
      stream_for @board
    else
      reject
    end
  end

  def unsubscribed
    # Broadcast that the user left
    BoardChannel.broadcast_to(@board, {
      type: "presence",
      user: current_user.username,
      status: "offline"
    }) if @board
  end
end
```

The `reject` method stops the subscription from being created. No stream is set up, and no data will be sent. On the client side, handle the rejection:

```javascript
consumer.subscriptions.create(
  { channel: "BoardChannel", board_id: 42 },
  {
    connected() {
      console.log("Successfully subscribed to board")
    },

    rejected() {
      console.log("Subscription rejected — you may not have access")
      // Show an error message or redirect
    },

    received(data) {
      // Handle board updates
    }
  }
)
```

You can also perform authorization checks in channel actions, not just in `subscribed`:

```ruby
def update_card(data)
  card = @board.cards.find(data["card_id"])

  unless current_user.can_edit?(card)
    transmit(error: "You do not have permission to edit this card")
    return
  end

  card.update!(title: data["title"])
  BoardChannel.broadcast_to(@board, {
    type: "card_updated",
    card_id: card.id,
    title: card.title
  })
end
```

## Common Pitfalls
1. **Authorizing only in `subscribed` but not in actions** — A user who was authorized at subscription time might lose access later (e.g., removed from a team). High-security actions should re-check permissions.
2. **Not calling `reject` and not calling `stream_from`** — If you skip both, the subscription is technically "established" but receives no data. This is confusing to debug. Always explicitly reject or stream.
3. **Leaking error details in rejection** — Unlike HTTP responses, `reject` does not carry a message. Do not try to pass error details through it. Use `transmit` before `reject` if you need to explain why access was denied.

## Best Practices
1. **Always authorize in `subscribed`** — Treat it like a `before_action` in a controller. Check that the user has access to the resource identified by `params`.
2. **Use `find_by` instead of `find`** — `find` raises an exception if the record does not exist, which would crash the subscription handler. `find_by` returns `nil`, letting you reject gracefully.
3. **Log rejected subscriptions** — Monitoring rejected subscription attempts helps detect abuse or misconfigured clients.

## Summary
- Call `reject` in `subscribed` to refuse unauthorized subscriptions — the client's `rejected` callback will fire.
- Always verify that `current_user` has permission to access the resource identified in `params`.
- Use `find_by` instead of `find` to avoid exceptions when resources do not exist.
- Re-check authorization in individual channel actions for high-security operations.
- The `reject` method does not carry error messages — use `transmit` before `reject` if you need to explain the denial.

## Code Examples

**Authorizing a subscription — only board members can subscribe, others are rejected**

```ruby
class BoardChannel < ApplicationCable::Channel
  def subscribed
    @board = Board.find_by(id: params[:board_id])

    if @board && current_user.member_of?(@board)
      stream_for @board
    else
      reject
    end
  end
end
```


## Resources

- [Action Cable - Channel Authorization](https://guides.rubyonrails.org/action_cable_overview.html#client-server-interactions) — Official guide on authorizing channel subscriptions

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*