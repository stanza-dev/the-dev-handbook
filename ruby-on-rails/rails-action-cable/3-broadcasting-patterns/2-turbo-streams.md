---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-turbo-streams"
---

# Broadcasting with Turbo Streams

Turbo Streams provide a higher-level way to broadcast HTML updates over Action Cable.

## What are Turbo Streams?

Turbo Streams send HTML fragments with instructions on how to update the page:

```html
<turbo-stream action="append" target="messages">
  <template>
    <div class="message">Hello!</div>
  </template>
</turbo-stream>
```

Actions: `append`, `prepend`, `replace`, `update`, `remove`, `before`, `after`

## Setup Turbo

```ruby
# Gemfile
gem "turbo-rails"
```

```bash
bin/rails turbo:install
```

## Broadcasting from Models

```ruby
class Message < ApplicationRecord
  belongs_to :room

  # Automatically broadcast on create/update/destroy
  broadcasts_to :room

  # Custom options
  broadcasts_to :room, inserts_by: :prepend, target: "messages"
end
```

This automatically:
1. Creates a stream subscription for the room
2. Broadcasts turbo-stream HTML when messages change
3. Updates the DOM on connected clients

## Manual Turbo Stream Broadcasting

```ruby
class Comment < ApplicationRecord
  after_create_commit :broadcast_comment

  private

  def broadcast_comment
    broadcast_append_to(
      article,
      target: "comments",
      partial: "comments/comment",
      locals: { comment: self }
    )
  end
end

# Other broadcast methods:
# broadcast_prepend_to
# broadcast_replace_to
# broadcast_update_to
# broadcast_remove_to
```

## View Setup

```erb
<!-- app/views/rooms/show.html.erb -->
<h1><%= @room.name %></h1>

<!-- Subscribe to broadcasts -->
<%= turbo_stream_from @room %>

<!-- Messages container that will receive broadcasts -->
<div id="messages">
  <%= render @room.messages %>
</div>

<%= form_with model: [@room, Message.new] do |f| %>
  <%= f.text_field :body %>
  <%= f.submit "Send" %>
<% end %>
```

## Broadcast Actions Comparison

```ruby
# Append to end of container
broadcast_append_to(room, target: "messages")

# Prepend to beginning
broadcast_prepend_to(room, target: "messages")

# Replace entire element
broadcast_replace_to(room)

# Update inner content only
broadcast_update_to(room, target: "message_#{id}")

# Remove element
broadcast_remove_to(room)
```

## Custom Broadcasts

```ruby
class Notification < ApplicationRecord
  after_create_commit :broadcast_notification

  private

  def broadcast_notification
    Turbo::StreamsChannel.broadcast_update_to(
      user,
      target: "notification_badge",
      html: "<span class='badge'>#{user.unread_notifications_count}</span>"
    )

    Turbo::StreamsChannel.broadcast_prepend_to(
      user,
      target: "notifications",
      partial: "notifications/notification",
      locals: { notification: self }
    )
  end
end
```

## Multiple Streams

```erb
<!-- Subscribe to multiple streams -->
<%= turbo_stream_from @room %>
<%= turbo_stream_from @room, :presence %>
<%= turbo_stream_from current_user, :notifications %>
```

## Resources

- [Turbo Streams](https://turbo.hotwired.dev/handbook/streams) — Turbo Streams documentation

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*