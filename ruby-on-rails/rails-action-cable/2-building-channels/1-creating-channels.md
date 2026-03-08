---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-creating-channels"
---

# Creating Your First Channel

## Introduction
Channels are the heart of Action Cable — they define how your server handles real-time subscriptions and messages. Creating a channel in Rails follows the same generator-driven workflow you already know from models and controllers, making it straightforward to add real-time features to any application.

## Key Concepts
- **Channel Generator**: The `rails generate channel` command creates the server-side channel class and client-side JavaScript subscription file.
- **`subscribed` Callback**: Called when a client subscribes to the channel. This is where you set up streams.
- **`unsubscribed` Callback**: Called when a client disconnects or explicitly unsubscribes. Use this for cleanup.
- **`stream_from`**: Connects the subscription to a named stream. Any data broadcast to that stream name will be delivered to this subscription.
- **`stream_for`**: A model-aware version of `stream_from` that generates a stream name from an ActiveRecord object.

## Real World Context
Imagine you are building a notifications feature. You need a NotificationsChannel that streams new notifications to the logged-in user. When a user opens your app, their browser subscribes to the channel. When another part of your system creates a notification (a background job, another controller action), it broadcasts to the user's stream, and the notification appears instantly — no page refresh needed.

## Deep Dive
Generate a channel using the Rails generator:

```bash
bin/rails generate channel Notifications
```

This creates two files:
- `app/channels/notifications_channel.rb` — server-side logic
- `app/javascript/channels/notifications_channel.js` — client-side subscription

Here is the server-side channel:

```ruby
# app/channels/notifications_channel.rb
class NotificationsChannel < ApplicationCable::Channel
  def subscribed
    # stream_for generates a unique stream name per user
    stream_for current_user
  end

  def unsubscribed
    # Clean up when the user disconnects
  end
end
```

The `stream_for current_user` call creates a stream name like `notifications:User#42`. Any broadcast sent to that stream reaches only that user's subscriptions.

For scenarios where you need a custom stream name (such as a chat room), use `stream_from`:

```ruby
class ChatChannel < ApplicationCable::Channel
  def subscribed
    room = params[:room]
    stream_from "chat_room_#{room}"
  end
end
```

The `params` hash contains data sent by the client when subscribing. This lets you create dynamic streams based on client input.

On the client side, create the subscription:

```javascript
import consumer from "./consumer"

consumer.subscriptions.create("NotificationsChannel", {
  connected() {
    console.log("Connected to notifications")
  },

  disconnected() {
    console.log("Disconnected from notifications")
  },

  received(data) {
    // Called when data is broadcast to this channel
    const notificationList = document.getElementById("notifications")
    notificationList.insertAdjacentHTML("beforeend",
      `<li>${data.message}</li>`
    )
  }
})
```

The `received` callback is where you handle incoming data on the client. This is the equivalent of handling a response in an HTTP request, but it fires automatically whenever the server broadcasts.

To subscribe with parameters (for example, a specific chat room):

```javascript
consumer.subscriptions.create(
  { channel: "ChatChannel", room: "general" },
  {
    received(data) {
      // Handle incoming chat message
    }
  }
)
```

## Common Pitfalls
1. **Forgetting to call `stream_from` or `stream_for` in `subscribed`** — Without setting up a stream, the subscription exists but never receives any data. The channel silently does nothing.
2. **Not validating `params` in `subscribed`** — Clients can send arbitrary parameters. Always validate that the user has access to the requested resource (e.g., check that the user is a member of the chat room).
3. **Mixing up `stream_from` and `stream_for`** — `stream_for` takes an object and generates a consistent stream name. `stream_from` takes a raw string. Using both for the same concept leads to mismatched stream names.

## Best Practices
1. **Use `stream_for` with ActiveRecord objects** — It generates consistent, collision-free stream names and works seamlessly with `broadcast_to`.
2. **Authorize in `subscribed`** — Check that `current_user` has permission to access the requested stream. Reject unauthorized subscriptions with `reject`.
3. **Keep the `received` callback thin** — On the client side, extract complex DOM manipulation into separate functions. The `received` callback should just dispatch, not contain business logic.

## Summary
- Use `bin/rails generate channel ChannelName` to scaffold both server and client files.
- The `subscribed` callback sets up streams; `unsubscribed` handles cleanup.
- `stream_for` generates stream names from ActiveRecord objects; `stream_from` uses raw string names.
- Client subscriptions are created with `consumer.subscriptions.create` and handle data in the `received` callback.
- Always authorize access in the `subscribed` callback and validate incoming parameters.

## Code Examples

**A channel that streams notifications to the current user — stream_for generates a unique stream name per user**

```ruby
# app/channels/notifications_channel.rb
class NotificationsChannel < ApplicationCable::Channel
  def subscribed
    stream_for current_user
  end

  def unsubscribed
    # Cleanup when user disconnects
  end
end
```


## Resources

- [Action Cable - Channels](https://guides.rubyonrails.org/action_cable_overview.html#server-side-components-channels) — Official guide on creating and configuring Action Cable channels

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*