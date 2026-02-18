---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-chat-channel"
---

# Building a Chat Room

Let's build a complete chat room feature with multiple rooms and real-time messages.

## The Chat Channel

```ruby
# app/channels/chat_channel.rb
class ChatChannel < ApplicationCable::Channel
  def subscribed
    @room = Room.find(params[:room_id])

    # Authorization check
    if @room.accessible_by?(current_user)
      stream_for @room
    else
      reject
    end
  end

  def unsubscribed
    # Notify others user left (optional)
    broadcast_user_left if @room
  end

  # Receive messages from client
  def speak(data)
    message = @room.messages.create!(
      user: current_user,
      body: data["message"]
    )

    # Broadcast to all subscribers
    ChatChannel.broadcast_to(@room, {
      type: "message",
      message: render_message(message)
    })
  end

  # Client can call this action
  def typing
    ChatChannel.broadcast_to(@room, {
      type: "typing",
      user: current_user.name
    })
  end

  private

  def render_message(message)
    ApplicationController.renderer.render(
      partial: "messages/message",
      locals: { message: message }
    )
  end

  def broadcast_user_left
    ChatChannel.broadcast_to(@room, {
      type: "user_left",
      user: current_user.name
    })
  end
end
```

## Client-Side JavaScript

```javascript
// app/javascript/channels/chat_channel.js
import consumer from "./consumer"

const chatChannel = {
  subscribe(roomId) {
    return consumer.subscriptions.create(
      { channel: "ChatChannel", room_id: roomId },
      {
        connected() {
          console.log(`Connected to room ${roomId}`)
        },

        disconnected() {
          console.log("Disconnected from chat")
        },

        received(data) {
          switch(data.type) {
            case "message":
              this.appendMessage(data.message)
              break
            case "typing":
              this.showTypingIndicator(data.user)
              break
            case "user_left":
              this.showUserLeft(data.user)
              break
          }
        },

        // Send a message
        speak(message) {
          this.perform("speak", { message: message })
        },

        // Notify typing
        typing() {
          this.perform("typing")
        },

        appendMessage(html) {
          const messages = document.getElementById("messages")
          messages.insertAdjacentHTML("beforeend", html)
          messages.scrollTop = messages.scrollHeight
        },

        showTypingIndicator(user) {
          const indicator = document.getElementById("typing-indicator")
          indicator.textContent = `${user} is typing...`
          clearTimeout(this.typingTimeout)
          this.typingTimeout = setTimeout(() => {
            indicator.textContent = ""
          }, 2000)
        },

        showUserLeft(user) {
          // Show system message
        }
      }
    )
  }
}

// Usage
const roomId = document.getElementById("chat-room").dataset.roomId
const subscription = chatChannel.subscribe(roomId)

// Send message on form submit
document.getElementById("message-form").addEventListener("submit", (e) => {
  e.preventDefault()
  const input = document.getElementById("message-input")
  subscription.speak(input.value)
  input.value = ""
})

// Typing indicator
document.getElementById("message-input").addEventListener("keypress", () => {
  subscription.typing()
})
```

## Message Partial

```erb
<!-- app/views/messages/_message.html.erb -->
<div class="message" id="message_<%= message.id %>">
  <div class="message-header">
    <strong><%= message.user.name %></strong>
    <time><%= message.created_at.strftime("%H:%M") %></time>
  </div>
  <div class="message-body">
    <%= message.body %>
  </div>
</div>
```

## Resources

- [Action Cable Example](https://guides.rubyonrails.org/action_cable_overview.html#full-stack-example) — Full chat room example

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*