---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-creating-channels"
---

# Creating Your First Channel

Channels handle WebSocket messages for specific features. Let's build a notifications channel.

## Generate a Channel

```bash
bin/rails generate channel Notifications
```

This creates:
- `app/channels/notifications_channel.rb`
- `app/javascript/channels/notifications_channel.js`

## Server-Side Channel

```ruby
# app/channels/notifications_channel.rb
class NotificationsChannel < ApplicationCable::Channel
  def subscribed
    # Called when client subscribes
    stream_for current_user
  end

  def unsubscribed
    # Called when client disconnects
    # Cleanup if needed
  end
end
```

### Streaming Methods

```ruby
class NotificationsChannel < ApplicationCable::Channel
  def subscribed
    # Stream from a named broadcast
    stream_from "notifications_#{current_user.id}"

    # OR stream for a model (creates stream name automatically)
    stream_for current_user
    # Equivalent to: stream_from "notifications:#{current_user.to_gid_param}"
  end
end
```

## Client-Side Subscription

```javascript
// app/javascript/channels/notifications_channel.js
import consumer from "./consumer"

consumer.subscriptions.create("NotificationsChannel", {
  connected() {
    // Called when WebSocket connection is established
    console.log("Connected to notifications!")
  },

  disconnected() {
    // Called when WebSocket connection is closed
    console.log("Disconnected from notifications")
  },

  received(data) {
    // Called when data is broadcast to this channel
    console.log("Received:", data)
    this.displayNotification(data)
  },

  displayNotification(data) {
    const container = document.getElementById("notifications")
    if (container) {
      const notification = document.createElement("div")
      notification.className = "notification"
      notification.textContent = data.message
      container.prepend(notification)
    }
  }
})
```

## Broadcasting Messages

Send notifications from anywhere in your app:

```ruby
# From a controller
class CommentsController < ApplicationController
  def create
    @comment = @article.comments.create(comment_params)

    # Notify the article author
    NotificationsChannel.broadcast_to(
      @article.author,
      {
        type: "new_comment",
        message: "#{current_user.name} commented on your article",
        url: article_path(@article)
      }
    )

    redirect_to @article
  end
end

# From a model callback
class Comment < ApplicationRecord
  after_create_commit :notify_author

  private

  def notify_author
    NotificationsChannel.broadcast_to(
      article.author,
      { message: "New comment on #{article.title}" }
    )
  end
end

# From a background job
class SendNotificationJob < ApplicationJob
  def perform(user, message)
    NotificationsChannel.broadcast_to(user, { message: message })
  end
end
```

## Alternative: Named Streams

```ruby
# Channel
class NotificationsChannel < ApplicationCable::Channel
  def subscribed
    stream_from "notifications_#{current_user.id}"
  end
end

# Broadcasting
ActionCable.server.broadcast(
  "notifications_#{user.id}",
  { message: "Hello!" }
)
```

## Resources

- [Action Cable Channels](https://guides.rubyonrails.org/action_cable_overview.html#channels) — Guide to creating channels

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*