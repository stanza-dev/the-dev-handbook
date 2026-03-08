---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-turbo-streams"
---

# Turbo Streams over Action Cable

## Introduction
Turbo Streams is the primary way Rails 8.1 applications deliver real-time DOM updates. When combined with Action Cable, Turbo Streams let you broadcast HTML fragments that automatically update the page without writing any custom JavaScript. This is the most Rails-idiomatic way to build real-time features.

## Key Concepts
- **Turbo Streams**: An HTML-over-the-wire approach where the server sends small HTML fragments with instructions (append, prepend, replace, remove, update) that Turbo applies to the DOM.
- **`turbo_stream_from`**: A view helper that creates a Turbo Streams subscription to an Action Cable channel, connecting DOM updates to a model's broadcasts.
- **`broadcast_append_to`**: A model method that broadcasts a Turbo Stream "append" action, adding HTML to a specified container.
- **`broadcasts_to`**: A model-level declaration that automatically broadcasts Turbo Stream updates on create, update, and destroy.

## Real World Context
Instead of writing JavaScript to receive a JSON payload, parse it, and manually insert HTML into the DOM, Turbo Streams let the server send ready-to-render HTML. When a new comment is posted, the server broadcasts a Turbo Stream that appends the rendered comment partial to the comments list. Every connected user sees the new comment appear — zero client-side JavaScript required.

## Deep Dive
First, set up the subscription in your view:

```erb
<%# app/views/rooms/show.html.erb %>
<%= turbo_stream_from @room %>

<div id="messages">
  <%= render @room.messages %>
</div>

<%= form_with(model: [@room, Message.new]) do |f| %>
  <%= f.text_field :body %>
  <%= f.submit "Send" %>
<% end %>
```

The `turbo_stream_from @room` helper generates a `<turbo-cable-stream-source>` element that subscribes to Action Cable and applies incoming Turbo Stream actions.

In the model, declare automatic broadcasts:

```ruby
# app/models/message.rb
class Message < ApplicationRecord
  belongs_to :room
  belongs_to :user

  broadcasts_to :room
end
```

The `broadcasts_to :room` declaration automatically broadcasts Turbo Stream updates when a message is created, updated, or destroyed. It renders the `_message.html.erb` partial and sends the HTML to all subscribers.

For more control, use explicit broadcast methods in callbacks:

```ruby
class Message < ApplicationRecord
  belongs_to :room
  belongs_to :user

  after_create_commit -> {
    broadcast_append_to room,
      target: "messages",
      partial: "messages/message",
      locals: { message: self }
  }

  after_update_commit -> {
    broadcast_replace_to room,
      target: self,
      partial: "messages/message",
      locals: { message: self }
  }

  after_destroy_commit -> {
    broadcast_remove_to room,
      target: self
  }
end
```

The available Turbo Stream actions are:

- `broadcast_append_to` — Adds content at the end of a container.
- `broadcast_prepend_to` — Adds content at the beginning of a container.
- `broadcast_replace_to` — Replaces an existing element entirely.
- `broadcast_update_to` — Updates the content inside an existing element.
- `broadcast_remove_to` — Removes an element from the DOM.

Each method specifies a `target` (the DOM id) and a `partial` (the HTML to render).

## Common Pitfalls
1. **Missing DOM target id** — Turbo Streams target elements by their `id` attribute. If the target id does not exist in the DOM, the stream action silently fails. Always ensure your container has the correct `id`.
2. **Using `broadcasts_to` without a partial** — The automatic broadcast renders `_model_name.html.erb` by convention. If this partial does not exist, broadcasts will fail with a missing template error.
3. **Broadcasting in `after_create` instead of `after_create_commit`** — Turbo Stream broadcasts must fire after the transaction commits. Use `_commit` callbacks to ensure data consistency.

## Best Practices
1. **Start with `broadcasts_to` for simple CRUD** — It handles create, update, and destroy automatically with zero configuration. Use explicit callbacks only when you need custom targets or conditions.
2. **Use `dom_id` for target ids** — Rails' `dom_id` helper generates consistent ids like `message_42`. Use it in both your partials and broadcast calls to avoid mismatches.
3. **Keep partials self-contained** — The partial rendered in a broadcast must work without the parent view's instance variables. Pass everything it needs via `locals`.

## Summary
- Turbo Streams deliver real-time DOM updates by broadcasting HTML fragments over Action Cable.
- `turbo_stream_from @model` in a view subscribes to broadcasts for that model.
- `broadcasts_to :association` automatically broadcasts on create, update, and destroy.
- Explicit methods (`broadcast_append_to`, `broadcast_replace_to`, etc.) give fine-grained control over what is sent and where it appears.
- Always ensure target DOM ids exist and use `after_create_commit` callbacks for data consistency.

## Code Examples

**The simplest Turbo Streams setup — broadcasts_to handles create, update, and destroy with zero custom JavaScript**

```ruby
class Message < ApplicationRecord
  belongs_to :room
  belongs_to :user

  # Automatically broadcast create/update/destroy as Turbo Streams
  broadcasts_to :room
end
```


## Resources

- [Turbo Streams Handbook](https://turbo.hotwired.dev/handbook/streams) — Official Turbo handbook section on Turbo Streams and their actions
- [Turbo Rails GitHub Repository](https://github.com/hotwired/turbo-rails) — Source code for the turbo-rails gem including broadcast helpers

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*