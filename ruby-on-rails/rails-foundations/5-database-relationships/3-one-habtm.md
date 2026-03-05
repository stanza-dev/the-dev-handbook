---
source_course: "rails-foundations"
source_lesson: "rails-foundations-has-one-habtm"
---

# Has One and HABTM Associations

## Introduction

Beyond `has_many` and `belongs_to`, Rails provides two more association types: `has_one` for strict one-to-one relationships and `has_and_belongs_to_many` (HABTM) for simple many-to-many joins without an intermediate model.

## Key Concepts

- **`has_one`**: Declares a one-to-one relationship where the other model holds the foreign key.
- **`has_and_belongs_to_many` (HABTM)**: A many-to-many relationship using a join table with no model.
- **`has_many :through`**: A many-to-many relationship using a join model — preferred over HABTM when you need attributes on the join.

## Real World Context

A user `has_one :profile`, an order `has_one :invoice`, a country `has_one :capital`. HABTM is used for simple tagging where the join has no data — but in most production apps, `has_many :through` is preferred because requirements grow.

## Deep Dive

### has_one

A `has_one` association means exactly one related record exists:

```ruby
class User < ApplicationRecord
  has_one :profile, dependent: :destroy
end

class Profile < ApplicationRecord
  belongs_to :user
end
```

The `profiles` table holds the `user_id` foreign key. The user does not have a `profile_id`.

```ruby
user = User.create(name: "Alice")
user.create_profile(bio: "Rails developer")
user.profile  # => #<Profile bio: "Rails developer"...>
```

Notice `create_profile` (singular) — `has_one` naming reflects the one-to-one nature.

### has_and_belongs_to_many (HABTM)

HABTM creates a direct many-to-many link with no model:

```ruby
class Article < ApplicationRecord
  has_and_belongs_to_many :categories
end

class Category < ApplicationRecord
  has_and_belongs_to_many :articles
end
```

The join table must combine both model names alphabetically:

```ruby
class CreateArticlesCategories < ActiveRecord::Migration[8.1]
  def change
    create_join_table :articles, :categories do |t|
      t.index [:article_id, :category_id]
      t.index [:category_id, :article_id]
    end
  end
end
```

### HABTM vs has_many :through

```ruby
# HABTM — no join model, no extra attributes
class Article < ApplicationRecord
  has_and_belongs_to_many :tags
end

# has_many :through — join model with attributes
class Article < ApplicationRecord
  has_many :taggings
  has_many :tags, through: :taggings
end
```

Use HABTM only when the join is truly attribute-free. The moment you need timestamps or ordering, switch to `has_many :through`.

## Common Pitfalls

- **Using HABTM when you need join attributes**: Migrating from HABTM to `has_many :through` later is painful.
- **Forgetting `dependent: :destroy` on has_one**: Deleting the parent leaves an orphaned record.
- **Wrong join table naming**: HABTM requires alphabetical order (`articles_categories`, not `categories_articles`).

## Best Practices

- Prefer `has_many :through` over HABTM for all new many-to-many relationships.
- Always add `dependent: :destroy` or `dependent: :nullify` to `has_one`.
- Use `create_join_table` in migrations for HABTM — it handles naming automatically.

## Summary

- `has_one` defines a one-to-one relationship where the related model holds the foreign key.
- HABTM creates many-to-many through a join table with no intermediate model.
- `has_many :through` is preferred over HABTM because it supports join attributes.
- Join tables for HABTM must be named alphabetically.
- Always use `dependent:` options to prevent orphaned records.

## Code Examples

**has_one for one-to-one relationships and has_many :through for many-to-many — note the singular create_profile method.**

```ruby
# has_one — one-to-one relationship
class User < ApplicationRecord
  has_one :profile, dependent: :destroy
end

user = User.first
user.create_profile(bio: "Rails developer")  # singular!
user.profile  # => #<Profile ...>

# has_many :through — preferred many-to-many
class Article < ApplicationRecord
  has_many :taggings
  has_many :tags, through: :taggings
end
```


## Resources

- [Active Record Associations](https://guides.rubyonrails.org/association_basics.html) — Official guide covering all association types including has_one and HABTM

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*