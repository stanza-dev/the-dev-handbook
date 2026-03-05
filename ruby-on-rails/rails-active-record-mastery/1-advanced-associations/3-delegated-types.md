---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-delegated-types"
---

# Delegated Types

## Introduction

Delegated types, introduced in Rails 6.1, offer a cleaner alternative to Single Table Inheritance when your types have very different attributes. Instead of one table with many nullable columns, each type gets its own table while sharing a common interface.

## Key Concepts

- **Delegated type**: A pattern where a base model delegates type-specific behavior to separate models, each with their own table.
- **`delegated_type`**: The Rails macro that sets up the delegation, combining polymorphism with a cleaner API.
- **Entryable**: A conventional name for the polymorphic interface. Any name works.

## Real World Context

Consider a messaging system with text messages, images, and videos. With STI, all types share one table — but images need `width`, `height`, and `url`, while text messages just need `body`. Delegated types give each type its own table while the parent provides the shared interface.

## Deep Dive

### The Problem with STI for Divergent Types

```ruby
# STI approach — one table with many NULLs
class Message < ApplicationRecord; end
class TextMessage < Message; end  # only uses body
class ImageMessage < Message; end  # only uses image_url, width, height
```

Each row has many NULL columns for attributes it does not use.

### Delegated Types Solution

```ruby
class Message < ApplicationRecord
  delegated_type :entryable, types: %w[TextEntry ImageEntry VideoEntry]
end

class TextEntry < ApplicationRecord
  has_one :message, as: :entryable, touch: true
end

class ImageEntry < ApplicationRecord
  has_one :message, as: :entryable, touch: true
end
```

Each type gets its own table with only the columns it needs. The `messages` table holds shared attributes plus `entryable_type` and `entryable_id`.

### Using Delegated Types

```ruby
message = Message.create!(entryable: TextEntry.new(body: "Hello!"))

message.text_entry?   # => true
message.image_entry?  # => false
message.entryable     # => #<TextEntry body: "Hello!">

Message.text_entries  # scope for all text messages
```

The `delegated_type` macro generates type-checking predicates, type-safe accessors, and scopes for each type.

### Migration

```ruby
class CreateMessages < ActiveRecord::Migration[8.1]
  def change
    create_table :messages do |t|
      t.references :entryable, polymorphic: true, null: false
      t.references :sender, null: false, foreign_key: { to_table: :users }
      t.timestamps
    end

    create_table :text_entries do |t|
      t.text :body, null: false
      t.timestamps
    end
  end
end
```

## Common Pitfalls

- **Using delegated types when STI suffices**: If all types share the same columns, STI is simpler.
- **Forgetting the polymorphic columns**: The parent needs `entryable_type` and `entryable_id`.
- **Not indexing the polymorphic columns**: Always add an index on `[entryable_type, entryable_id]`.

## Best Practices

- Use delegated types when types have significantly different attributes.
- Keep shared attributes on the parent and type-specific attributes on the delegate.
- Use the generated scopes (`Message.text_entries`) for type-filtered queries.

## Summary

- Delegated types split type-specific attributes into separate tables while maintaining a shared parent.
- Use `delegated_type :entryable, types: [...]` on the parent model.
- Each delegate model has `has_one :parent, as: :entryable`.
- Prefer delegated types over STI when types have very different attributes.

## Code Examples

**Delegated types — each type gets its own table while sharing the Message parent model.**

```ruby
class Message < ApplicationRecord
  delegated_type :entryable, types: %w[TextEntry ImageEntry VideoEntry]
end

class TextEntry < ApplicationRecord
  has_one :message, as: :entryable, touch: true
end

message = Message.create!(entryable: TextEntry.new(body: "Hello!"))
message.text_entry?     # => true
message.entryable       # => #<TextEntry body: "Hello!">
Message.text_entries    # Scope for all text messages
```


## Resources

- [Active Record - Delegated Types](https://api.rubyonrails.org/classes/ActiveRecord/DelegatedType.html) — API documentation for delegated_type macro

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*