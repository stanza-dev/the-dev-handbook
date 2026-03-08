---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-broadcast-from-models"
---

# Broadcasting from Anywhere

## Introduction
Broadcasting is how your Rails application pushes data to connected clients. The power of Action Cable lies in the fact that you can broadcast from anywhere in your codebase — controllers, models, background jobs, or even the Rails console. Understanding where and how to broadcast is key to building responsive real-time features.

## Key Concepts
- **`broadcast_to`**: A class method on channels that sends data to all subscribers of a stream derived from a model. It pairs with `stream_for` in the channel.
- **`ActionCable.server.broadcast`**: A lower-level method that broadcasts to a raw stream name. It pairs with `stream_from` in the channel.
- **Broadcasting from Jobs**: The recommended pattern for heavy operations — broadcast the result from a background job to avoid blocking the request cycle.
- **Global Broadcasting**: Any code with access to the `ActionCable.server` object can broadcast, including rake tasks, console sessions, and service objects.

## Real World Context
In a project management app, when a user creates a task, the controller saves it to the database and enqueues a background job. The job processes the task (sends emails, updates counters), then broadcasts the new task to all team members viewing the board. This decouples the HTTP response from the real-time update, keeping both fast.

## Deep Dive
The most common way to broadcast is using `broadcast_to` on a channel class:

```ruby
# From a controller
class MessagesController < ApplicationController
  def create
    @room = Room.find(params[:room_id])
    @message = @room.messages.create!(message_params.merge(user: current_user))

    # Broadcast to all subscribers of ChatChannel for this room
    ChatChannel.broadcast_to(@room, {
      type: "new_message",
      id: @message.id,
      user: current_user.username,
      body: @message.body,
      created_at: @message.created_at.iso8601
    })

    head :created
  end
end
```

`ChatChannel.broadcast_to(@room, data)` generates the same stream name that `stream_for @room` uses inside the channel, ensuring the data reaches the right subscribers.

For broadcasting from a background job:

```ruby
# app/jobs/notification_broadcast_job.rb
class NotificationBroadcastJob < ApplicationJob
  queue_as :default

  def perform(notification)
    NotificationsChannel.broadcast_to(
      notification.user,
      {
        type: "notification",
        id: notification.id,
        title: notification.title,
        body: notification.body,
        read: false,
        created_at: notification.created_at.iso8601
      }
    )
  end
end
```

Trigger this from a model callback or service object:

```ruby
# app/models/notification.rb
class Notification < ApplicationRecord
  after_create_commit -> { NotificationBroadcastJob.perform_later(self) }
end
```

Using `after_create_commit` ensures the broadcast only fires after the database transaction commits, so subscribers always see consistent data.

You can also broadcast from the Rails console for debugging:

```ruby
# In rails console
user = User.find(1)
NotificationsChannel.broadcast_to(user, { type: "test", message: "Hello from console!" })
```

For raw stream names (when using `stream_from` instead of `stream_for`):

```ruby
ActionCable.server.broadcast("chat_room_general", {
  user: "System",
  body: "Welcome to the general channel!"
})
```

## Common Pitfalls
1. **Broadcasting before the transaction commits** — Using `after_create` instead of `after_create_commit` can broadcast data that gets rolled back if the transaction fails. Always use the `_commit` variants.
2. **Broadcasting large payloads** — Sending entire ActiveRecord objects or deeply nested associations through WebSockets wastes bandwidth and can expose sensitive attributes. Serialize only the fields the client needs.
3. **Blocking the request cycle with broadcasts** — If broadcasting involves any I/O (like fetching additional data), move it to a background job. The HTTP response should not wait for the broadcast to complete.

## Best Practices
1. **Use `after_create_commit` for model-triggered broadcasts** — This ensures data consistency by broadcasting only after the database transaction has committed.
2. **Move complex broadcasts to background jobs** — Keep controllers and model callbacks thin. Jobs can safely perform additional queries or formatting.
3. **Include a `type` field in every broadcast payload** — This lets the client-side `received` callback route messages to the correct handler.

## Summary
- `broadcast_to(model, data)` sends data to all subscribers of the stream derived from the model.
- Broadcasting works from controllers, models, background jobs, services, and even the Rails console.
- Always use `after_create_commit` (not `after_create`) to avoid broadcasting uncommitted data.
- Move heavy broadcast logic to background jobs to keep the request cycle fast.
- Serialize only the fields the client needs — never broadcast entire ActiveRecord objects.

## Code Examples

**The recommended pattern — model callback triggers a background job, which broadcasts to the user's channel**

```ruby
# Broadcasting from a model callback via a background job
class Notification < ApplicationRecord
  after_create_commit -> { NotificationBroadcastJob.perform_later(self) }
end

# app/jobs/notification_broadcast_job.rb
class NotificationBroadcastJob < ApplicationJob
  def perform(notification)
    NotificationsChannel.broadcast_to(
      notification.user,
      { type: "notification", title: notification.title }
    )
  end
end
```


## Resources

- [Action Cable - Broadcasting](https://guides.rubyonrails.org/action_cable_overview.html#client-server-interactions-broadcastings) — Official guide on broadcasting data to Action Cable streams

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*