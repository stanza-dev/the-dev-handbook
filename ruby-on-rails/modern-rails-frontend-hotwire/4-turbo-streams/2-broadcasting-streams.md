---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-broadcasting-streams"
---

# Broadcasting with Action Cable

## Introduction
Turbo Streams can be broadcast to all connected clients via Action Cable (WebSockets). When a model changes, the server pushes HTML fragments to every browser viewing the relevant page — enabling real-time features with minimal code.

## Key Concepts
- **`broadcasts_to`**: A model callback that automatically broadcasts stream actions when records are created, updated, or destroyed.
- **`turbo_stream_from`**: A view helper that subscribes a page to a broadcast channel.
- **Channel**: An Action Cable channel identified by a model or string that clients subscribe to.
- **Broadcastable**: The module (`Turbo::Broadcastable`) mixed into ActiveRecord models for broadcast support.

## Real World Context
Real-time broadcasts power chat applications, live comment feeds, collaborative editing, notification systems, and dashboards that update across all connected users. Without Turbo Streams, building these features requires custom WebSocket code, JSON serialization, and client-side rendering.

## Deep Dive
### Model Setup

Add broadcast callbacks to your model:

```ruby
class Message < ApplicationRecord
  belongs_to :chat

  broadcasts_to :chat
end
```

`broadcasts_to :chat` tells Rails: when a message is created, updated, or destroyed, broadcast the appropriate Turbo Stream action to the chat's stream. The broadcast happens asynchronously via Active Job.

This single line replaces what would be dozens of lines of custom WebSocket, JSON, and JavaScript code.

### View Subscription

Subscribe a page to receive broadcasts:

```erb
<!-- chats/show.html.erb -->
<%= turbo_stream_from @chat %>

<div id="messages">
  <%= render @chat.messages %>
</div>

<%= render "messages/form", message: Message.new(chat: @chat) %>
```

`turbo_stream_from @chat` generates a `<turbo-cable-stream-source>` element that opens a WebSocket connection to the chat's channel. Any broadcast to this channel will be received and applied.

### What Gets Broadcast

With `broadcasts_to :chat`, Rails automatically broadcasts:

- **Create**: `turbo_stream.append "messages", @message`
- **Update**: `turbo_stream.replace dom_id(@message), @message`
- **Destroy**: `turbo_stream.remove dom_id(@message)`

The HTML is rendered server-side using the model's partial (`messages/_message.html.erb`) and sent to all subscribers.

### Custom Broadcasts

For more control, broadcast manually:

```ruby
class Notification < ApplicationRecord
  after_create_commit -> {
    broadcast_prepend_to user,
      target: "notifications",
      partial: "notifications/notification"
  }

  after_destroy_commit -> {
    broadcast_remove_to user
  }
end
```

Manual broadcasts let you choose the action (prepend vs append), the target, and the partial.

### Morph Broadcasts (Turbo 8)

Broadcast with morph for state-preserving updates:

```ruby
after_update_commit -> {
  broadcast_morph_to :dashboard,
    target: dom_id(self),
    partial: "products/product"
}
```

Clients receive a morph stream that diffs the current and new HTML, preserving focus, scroll, and form state.

## Common Pitfalls
1. **Missing `turbo_stream_from` in the view** — Without the subscription helper, the page won't receive any broadcasts even if the model is configured.
2. **Broadcast rendering in wrong context** — Broadcasts render partials without a request context. Avoid using `current_user` or request-specific helpers in broadcast partials.

## Best Practices
1. **Use `broadcasts_to` for simple CRUD** — It handles create/update/destroy with sensible defaults.
2. **Use manual broadcasts for complex logic** — When you need custom actions, targets, or conditional broadcasting.

## Summary
- `broadcasts_to` on a model automatically sends Turbo Streams on create/update/destroy.
- `turbo_stream_from` in views subscribes the page to a channel via WebSockets.
- Broadcasts render server-side HTML and push it to all connected clients.
- Manual broadcast methods give full control over actions and targets.
- Morph broadcasts preserve DOM state for smooth live updates.

## Code Examples

**Complete real-time comments setup — one model line + one view helper enables live updates for all connected users**

```ruby
class Comment < ApplicationRecord
  belongs_to :post
  broadcasts_to :post
  # On create: appends to "comments" in all browsers viewing the post
  # On update: replaces the comment in all browsers
  # On destroy: removes the comment from all browsers
end

# View subscription:
# <%= turbo_stream_from @post %>
# <div id="comments">
#   <%= render @post.comments %>
# </div>
```


## Resources

- [Turbo Streams Broadcasts](https://github.com/hotwired/turbo-rails#broadcasting) — turbo-rails README section on broadcasting with Action Cable

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*