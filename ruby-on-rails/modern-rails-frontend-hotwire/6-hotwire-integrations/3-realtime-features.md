---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-realtime-features"
---

# Building Real-Time Features

## Introduction
Hotwire makes building real-time features straightforward by combining Turbo Streams broadcasting with Action Cable. You can build live chat, notifications, dashboards, and collaborative editing with a few lines of Ruby.

## Key Concepts
- **Action Cable**: Rails' WebSocket framework that powers real-time communication.
- **Subscription**: A client subscribes to a channel and receives broadcasts.
- **Broadcast Rendering**: Server renders HTML partials and broadcasts them as Turbo Streams.
- **Page Refresh Morphing**: Turbo 8 can broadcast a full page morph that intelligently updates all changed content.

## Real World Context
Real-time features are expected in modern applications: live notifications, collaborative document editing, real-time dashboards, chat systems, and live activity feeds. Before Hotwire, these required custom WebSocket handlers, JavaScript rendering code, and complex state synchronization. With Hotwire, you write Ruby and ERB.

## Deep Dive
### Live Notifications

```ruby
class Notification < ApplicationRecord
  belongs_to :user

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

The model broadcasts when notifications are created or destroyed. The partial is rendered server-side and pushed to the user's browser.

The view subscribes:

```erb
<%= turbo_stream_from current_user %>
<div id="notifications">
  <%= render @notifications %>
</div>
```

New notifications appear instantly for the user without polling or manual WebSocket code.

### Page Refresh Morphing (Turbo 8)

Turbo 8 introduced page refresh morphing — instead of broadcasting individual stream actions, you can broadcast a page morph that intelligently diffs and updates the entire page:

```erb
<!-- app/views/layouts/application.html.erb -->
<head>
  <meta name="turbo-refresh-method" content="morph">
  <meta name="turbo-refresh-scroll" content="preserve">
</head>
```

With these meta tags, Turbo will morph the page on refresh instead of replacing it, preserving scroll position, form inputs, and focus state.

```ruby
# Broadcasting a page refresh to all subscribers
Turbo::StreamsChannel.broadcast_refresh_to(@project)
```

This triggers all subscribed browsers to reload and morph the page. The server renders the full page, Turbo diffs it against the current DOM, and only the changed elements update.

### Live Dashboard

```ruby
class Metric < ApplicationRecord
  belongs_to :dashboard

  after_update_commit -> {
    broadcast_morph_to dashboard,
      target: dom_id(self),
      partial: "metrics/metric"
  }
end
```

Dashboard metrics update in real-time using morph broadcasts. Because morph preserves DOM state, animations, transitions, and user interactions are not interrupted.

## Common Pitfalls
1. **Broadcasting N+1 queries** — Broadcast partials can trigger N+1 queries since they render outside the request context. Use `includes` in the model scope or cache the associations.
2. **Not scoping broadcasts** — Broadcasting to all users when only specific users should see the update. Always scope broadcasts to the appropriate channel (user, project, team).

## Best Practices
1. **Use morph for dashboard-style updates** — When multiple elements change, morph broadcasts are simpler than individual stream actions.
2. **Scope broadcasts narrowly** — Broadcast to the specific user, project, or team, not globally.

## Summary
- Turbo Streams broadcasting with Action Cable enables real-time features with minimal code.
- Model callbacks like `broadcast_prepend_to` push updates to all subscribers.
- Turbo 8 page refresh morphing diffs the entire page, preserving DOM state.
- Morph broadcasts are ideal for dashboards and collaborative editing.
- Always scope broadcasts to the appropriate channel.

## Code Examples

**A complete real-time notification system — create broadcasts prepend, update broadcasts morph, destroy broadcasts remove**

```ruby
# Complete real-time notification system
class Notification < ApplicationRecord
  belongs_to :user

  after_create_commit -> {
    broadcast_prepend_to user,
      target: "notifications",
      partial: "notifications/notification"
  }

  after_update_commit -> {
    broadcast_morph_to user,
      target: dom_id(self),
      partial: "notifications/notification"
  }

  after_destroy_commit -> {
    broadcast_remove_to user
  }
end

# View: <%= turbo_stream_from current_user %>
# <div id="notifications"><%= render @notifications %></div>
```


## Resources

- [Action Cable Overview](https://guides.rubyonrails.org/action_cable_overview.html) — Rails guide on Action Cable, the WebSocket framework powering Turbo broadcasts

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*