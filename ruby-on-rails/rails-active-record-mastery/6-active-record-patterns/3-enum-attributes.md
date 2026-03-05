---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-enum-attributes"
---

# Enum Attributes

## Introduction

Enums let you define a set of named values for an attribute, backed by integers in the database. Instead of storing strings and worrying about typos, enums give you type-safe constants, query scopes, and predicate methods — all generated automatically.

## Key Concepts

- **`enum`**: A class method mapping named values to integers, providing scopes, predicates, and bang methods.
- **Integer-backed**: Enum values are stored as integers for efficient storage and indexing.
- **Hash syntax**: Rails 7+ requires `enum :status, { draft: 0, published: 1 }`.

## Real World Context

Almost every model has a status-like attribute: order status, article state, user role. Enums are the standard Rails approach. They replace magic strings with a clean API.

## Deep Dive

### Defining Enums

Rails 7+ uses hash syntax with keyword arguments:

```ruby
class Article < ApplicationRecord
  enum :status, { draft: 0, published: 1, archived: 2 }
  enum :category, { tech: 0, science: 1, business: 2 }, prefix: true
end
```

The integers are stored in the database. Always use explicit mappings.

### Generated Methods

```ruby
article = Article.new(status: :draft)

article.draft?      # => true
article.published?  # => false
article.published!  # Sets to published and saves

Article.draft       # Scope: all draft articles
Article.published   # Scope: all published articles
Article.statuses    # => {"draft"=>0, "published"=>1, "archived"=>2}
```

With `prefix: true`, methods become: `article.category_tech?`.

### Migration

```ruby
class AddStatusToArticles < ActiveRecord::Migration[8.1]
  def change
    add_column :articles, :status, :integer, default: 0, null: false
    add_index :articles, :status
  end
end
```

The `default: 0` corresponds to `:draft`. Always add an index.

### Prefix and Suffix Options

```ruby
class User < ApplicationRecord
  enum :role, { admin: 0, moderator: 1, member: 2 }, prefix: :role
  enum :status, { active: 0, archived: 1 }, suffix: :status
end

user.role_admin?      # => true
user.active_status?   # => true
```

Prefixes and suffixes prevent method name collisions.

### Validation

```ruby
class Article < ApplicationRecord
  enum :status, { draft: 0, published: 1, archived: 2 }, validate: true
end

article = Article.new(status: :invalid)
article.valid?  # => false (instead of raising ArgumentError)
```

## Common Pitfalls

- **Using array syntax**: `enum status: [:draft, :published]` is deprecated in Rails 7+. Use hash syntax.
- **Adding values in the middle**: New values must go at the end with the next integer. Inserting in the middle shifts existing records' meanings.
- **No database default**: Without a default, new records have NULL status.

## Best Practices

- Always use explicit integer mappings (hash syntax).
- Add new enum values at the end, never in the middle.
- Use `prefix: true` when enum names might clash.
- Add an index on enum columns.

## Summary

- Enums map named values to integers, providing predicates, bang methods, and scopes.
- Rails 7+ requires hash syntax: `enum :status, { draft: 0, published: 1 }`.
- Use `prefix:` or `suffix:` to avoid method name collisions.
- Always add new values at the end with the next integer.
- Add a database default and index on enum columns.

## Code Examples

**Enum attributes generate predicate methods, bang methods, and query scopes from a single declaration.**

```ruby
class Article < ApplicationRecord
  enum :status, { draft: 0, published: 1, archived: 2 }
end

article = Article.new(status: :draft)
article.draft?      # => true
article.published!  # Updates to published and saves

Article.published   # Scope: all published articles
Article.statuses    # => {"draft"=>0, "published"=>1, "archived"=>2}
```


## Resources

- [Active Record Enums](https://api.rubyonrails.org/classes/ActiveRecord/Enum.html) — API documentation for Active Record enum declarations

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*