---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-broadcast-serialization"
---

# Broadcast Payloads and Serialization

## Introduction
What you send through Action Cable matters just as much as how you send it. Poorly structured payloads waste bandwidth, leak sensitive data, and create fragile client-side code. This lesson covers how to design clean, efficient broadcast payloads that serve your frontend well.

## Key Concepts
- **Payload Serialization**: The process of converting Ruby objects into JSON for transmission over WebSockets. Action Cable automatically serializes hashes to JSON.
- **Selective Serialization**: Including only the fields the client needs, rather than dumping entire model attributes.
- **Payload Schema**: A consistent structure for broadcast messages that includes a type identifier and the relevant data.

## Real World Context
Broadcasting `user.as_json` sends every column — including `encrypted_password`, `email`, and `created_at` — to every connected client. Besides the security risk, it wastes bandwidth and forces the client to sift through irrelevant fields. Production applications define explicit payload shapes, much like API serializers for HTTP endpoints.

## Deep Dive
A well-structured broadcast payload has a clear schema:

```ruby
# Good: explicit, typed payload
ChatChannel.broadcast_to(room, {
  type: "new_message",
  data: {
    id: message.id,
    body: message.body,
    user: {
      id: message.user.id,
      username: message.user.username,
      avatar_url: message.user.avatar_url
    },
    created_at: message.created_at.iso8601
  }
})
```

Compare this with a lazy approach:

```ruby
# Bad: leaks data, wastes bandwidth
ChatChannel.broadcast_to(room, message.as_json(include: :user))
```

For reusable payload construction, create a broadcast builder or use a serializer:

```ruby
# app/builders/message_broadcast_builder.rb
class MessageBroadcastBuilder
  def self.call(message)
    {
      type: "new_message",
      data: {
        id: message.id,
        body: message.body,
        user: {
          id: message.user.id,
          username: message.user.username,
          avatar_url: message.user.avatar_url
        },
        created_at: message.created_at.iso8601
      }
    }
  end
end

# Usage
ChatChannel.broadcast_to(room, MessageBroadcastBuilder.call(message))
```

This pattern keeps payload definitions in one place, making it easy to update the structure when the frontend needs change.

When working with timestamps, always use ISO 8601 format:

```ruby
# Good: unambiguous, timezone-aware
created_at: message.created_at.iso8601
# Output: "2026-03-06T14:30:00Z"

# Bad: locale-dependent, ambiguous
created_at: message.created_at.to_s
# Output: "2026-03-06 14:30:00 +0000"
```

## Common Pitfalls
1. **Using `as_json` or `to_json` without filtering** — These methods serialize all model attributes by default, including sensitive fields. Always specify which fields to include.
2. **Inconsistent payload shapes** — If some broadcasts include a `type` field and others do not, client-side routing logic breaks. Establish a convention and follow it everywhere.
3. **Sending timestamps in local time** — Local time formats vary by server locale and are ambiguous. Always use `iso8601` for timestamps in broadcasts.

## Best Practices
1. **Create broadcast builder objects** — Centralize payload construction in dedicated classes. This ensures consistency and makes changes easy.
2. **Always include a `type` field** — Every broadcast should identify its purpose so the client can route it appropriately.
3. **Use ISO 8601 for all timestamps** — It is unambiguous, timezone-aware, and natively parsed by JavaScript's `Date` constructor.

## Summary
- Never broadcast raw ActiveRecord objects — serialize only the fields the client needs.
- Structure every payload with a `type` field and a `data` object for consistency.
- Use broadcast builder objects to centralize and standardize payload construction.
- Always format timestamps as ISO 8601 strings.
- Treat broadcast payloads with the same care as API responses — they are a public contract with your frontend.

## Code Examples

**A broadcast builder that produces a clean, typed payload with only the fields the client needs**

```ruby
class MessageBroadcastBuilder
  def self.call(message)
    {
      type: "new_message",
      data: {
        id: message.id,
        body: message.body,
        user: {
          id: message.user.id,
          username: message.user.username
        },
        created_at: message.created_at.iso8601
      }
    }
  end
end
```


## Resources

- [Active Model Serialization](https://api.rubyonrails.org/classes/ActiveModel/Serialization.html) — Rails API docs on model serialization methods including as_json

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*