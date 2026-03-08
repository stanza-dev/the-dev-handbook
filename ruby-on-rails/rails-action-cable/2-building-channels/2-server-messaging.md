---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-client-server-messaging"
---

# Client-Server Messaging

## Introduction
Action Cable channels are not just one-way pipes — clients can send messages to the server by invoking channel actions, and the server can respond or broadcast to others. This bidirectional messaging is what makes features like chat, typing indicators, and live forms possible.

## Key Concepts
- **Channel Actions**: Public methods on a channel class that clients can invoke via `perform`. They act like controller actions but for WebSocket messages.
- **`perform` Method**: The client-side method that sends a message to a specific channel action on the server.
- **`transmit` Method**: Sends data only to the subscription that triggered the action, not to all subscribers on the stream.
- **Broadcasting**: Sends data to all subscribers of a stream. Unlike `transmit`, which targets one subscription, broadcasting reaches everyone listening.

## Real World Context
In a chat application, when a user types a message and hits send, the client calls `perform('speak', { body: 'Hello!' })`. The server's `speak` action receives the message, saves it to the database, and broadcasts it to all subscribers in the room. Every connected user sees the new message instantly without any polling.

## Deep Dive
To define a server action that clients can call, add a public method to your channel:

```ruby
# app/channels/chat_channel.rb
class ChatChannel < ApplicationCable::Channel
  def subscribed
    @room = Room.find(params[:room_id])
    stream_for @room
  end

  # Clients call this via perform('speak', { body: '...' })
  def speak(data)
    message = @room.messages.create!(
      user: current_user,
      body: data["body"]
    )

    ChatChannel.broadcast_to(@room, {
      user: current_user.username,
      body: message.body,
      created_at: message.created_at.iso8601
    })
  end

  def typing(data)
    # Broadcast typing indicator to everyone except the sender
    ChatChannel.broadcast_to(@room, {
      type: "typing",
      user: current_user.username,
      is_typing: data["is_typing"]
    })
  end
end
```

Notice how `speak` receives a `data` hash with the parameters sent by the client. The action persists the message and broadcasts it to all subscribers.

On the client side, calling these actions looks like this:

```javascript
const chatChannel = consumer.subscriptions.create(
  { channel: "ChatChannel", room_id: 42 },
  {
    received(data) {
      if (data.type === "typing") {
        this.showTypingIndicator(data.user, data.is_typing)
      } else {
        this.appendMessage(data)
      }
    },

    speak(body) {
      // perform sends the action name and data to the server
      this.perform('speak', { body: body })
    },

    startTyping() {
      this.perform('typing', { is_typing: true })
    },

    stopTyping() {
      this.perform('typing', { is_typing: false })
    },

    appendMessage(data) {
      const messages = document.getElementById('messages')
      messages.insertAdjacentHTML('beforeend',
        `<div class="message"><strong>${data.user}:</strong> ${data.body}</div>`
      )
    },

    showTypingIndicator(user, isTyping) {
      const indicator = document.getElementById('typing-indicator')
      indicator.textContent = isTyping ? `${user} is typing...` : ''
    }
  }
)
```

The `this.perform(action, data)` call sends a message through the WebSocket to the named action on the server-side channel. The server processes it and can respond by broadcasting or transmitting.

If you need to respond to only the caller (not all subscribers), use `transmit`:

```ruby
def fetch_history(data)
  messages = @room.messages.order(created_at: :desc).limit(50)
  transmit(messages: messages.map { |m| { user: m.user.username, body: m.body } })
end
```

The `transmit` method sends data only to the subscription that invoked the action, making it ideal for queries that should not be broadcast to everyone.

## Common Pitfalls
1. **Using `transmit` when you mean `broadcast_to`** — `transmit` only sends to the calling subscription. If you want all users in a chat room to see a new message, you must broadcast, not transmit.
2. **Not handling different message types in `received`** — When a channel broadcasts multiple types of data (messages, typing indicators, presence updates), the `received` callback must differentiate between them, typically using a `type` field.
3. **Performing database writes without error handling** — If `create!` raises an exception in a channel action, the error is logged but the client gets no feedback. Wrap writes in error handling and transmit errors back to the caller.

## Best Practices
1. **Use `broadcast_to` over `broadcast`** — `broadcast_to(model, data)` generates consistent stream names that match `stream_for`. Using raw `ActionCable.server.broadcast` with hand-typed names is error-prone.
2. **Add a `type` field to broadcast payloads** — This lets the client-side `received` callback route messages to the correct handler function.
3. **Debounce typing indicators on the client** — Sending a `typing` action on every keystroke floods the WebSocket. Debounce the call so it fires at most once every 500 milliseconds.

## Summary
- Channel actions are public methods that clients invoke with `perform('action_name', data)`.
- `broadcast_to` sends data to all subscribers of a stream; `transmit` sends only to the calling subscription.
- The client's `received` callback should route incoming data by type to separate handler functions.
- Always validate and authorize data in channel actions, just as you would in controller actions.
- Debounce high-frequency client actions like typing indicators to avoid flooding the WebSocket.

## Code Examples

**A channel action that persists a chat message and broadcasts it to all room subscribers**

```ruby
class ChatChannel < ApplicationCable::Channel
  def speak(data)
    message = @room.messages.create!(
      user: current_user,
      body: data["body"]
    )
    ChatChannel.broadcast_to(@room, {
      user: current_user.username,
      body: message.body
    })
  end
end
```


## Resources

- [Action Cable - Client-Server Interactions](https://guides.rubyonrails.org/action_cable_overview.html#client-server-interactions) — Official guide on sending messages between client and server via channels

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*