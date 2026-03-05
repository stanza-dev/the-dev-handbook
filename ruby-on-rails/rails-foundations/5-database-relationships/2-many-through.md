---
source_course: "rails-foundations"
source_lesson: "rails-foundations-has-many-through"
---

# Many-to-Many with has_many :through

## Introduction

When two models need to relate to each other in both directions, you need a many-to-many relationship. Rails handles this elegantly with `has_many :through`, using a join model to connect the two sides.

## Key Concepts

- **Many-to-many relationship**: A relationship where both sides can have multiple associated records (e.g., articles have many tags, tags have many articles).
- **Join model**: An intermediate model (e.g., `ArticleTag`) that holds the foreign keys for both sides.
- **`has_many :through`**: Declares a many-to-many association that traverses through a join model.
- **`has_and_belongs_to_many` (HABTM)**: A simpler but less flexible alternative that doesn't use a join model.

## Real World Context

Many-to-many relationships appear everywhere: students enroll in courses, users have roles, products belong to categories. `has_many :through` is preferred over HABTM because the join model can hold additional attributes (e.g., enrollment date, role permissions) and supports validations and callbacks.

## Deep Dive

### The Models

```ruby
class Article < ApplicationRecord
  has_many :article_tags
  has_many :tags, through: :article_tags
end

class Tag < ApplicationRecord
  has_many :article_tags
  has_many :articles, through: :article_tags
end

class ArticleTag < ApplicationRecord
  belongs_to :article
  belongs_to :tag
end
```

### The Migrations

```ruby
class CreateTags < ActiveRecord::Migration[8.1]
  def change
    create_table :tags do |t|
      t.string :name
      t.timestamps
    end
  end
end

class CreateArticleTags < ActiveRecord::Migration[8.1]
  def change
    create_table :article_tags do |t|
      t.references :article, null: false, foreign_key: true
      t.references :tag, null: false, foreign_key: true
      t.timestamps
    end
    add_index :article_tags, [:article_id, :tag_id], unique: true
  end
end
```

### Using the Association

```ruby
ruby_tag = Tag.create(name: "Ruby")
rails_tag = Tag.create(name: "Rails")

article = Article.create(title: "Learning Rails")
article.tags << ruby_tag
article.tags << rails_tag

article.tags          # => [ruby_tag, rails_tag]
ruby_tag.articles     # => [article]
article.tag_ids = [1, 2, 3]  # Assign by IDs
```

### Adding Attributes to the Join

```ruby
class ArticleTag < ApplicationRecord
  belongs_to :article
  belongs_to :tag
  # Additional column: position
end

article.article_tags.create(tag: ruby_tag, position: 1)
```

### When to Use `has_many :through` vs HABTM

Use `has_many :through` when you need:
- Additional attributes on the join
- Validations or callbacks on the join
- To query the join model directly

## Common Pitfalls

- **Forgetting the unique index**: Without `add_index :article_tags, [:article_id, :tag_id], unique: true`, duplicate associations can be created.
- **Missing the join model declaration**: Both sides need `has_many :article_tags` in addition to the `:through` association.
- **Using HABTM when you need join attributes**: Start with `has_many :through` since you can always add attributes to the join model later.

## Best Practices

- Always use `has_many :through` over HABTM for new applications.
- Add a unique composite index on the join table to prevent duplicates.
- Declare the join model association (`has_many :article_tags`) before the through association.

## Summary

- `has_many :through` creates many-to-many relationships using a join model.
- The join model holds foreign keys for both sides and can have additional attributes.
- Both sides declare `has_many :join_model` and `has_many :other_model, through: :join_model`.
- Add a unique composite index to prevent duplicate associations.
- Prefer `has_many :through` over HABTM for flexibility.

## Code Examples

**A many-to-many relationship using has_many :through. The ArticleTag join model connects articles and tags, allowing both sides to access the other.**

```ruby
class Article < ApplicationRecord
  has_many :article_tags
  has_many :tags, through: :article_tags
end

class Tag < ApplicationRecord
  has_many :article_tags
  has_many :articles, through: :article_tags
end

class ArticleTag < ApplicationRecord
  belongs_to :article
  belongs_to :tag
end
```


## Resources

- [Active Record Associations - has_many :through](https://guides.rubyonrails.org/association_basics.html#the-has-many-through-association) — Guide to many-to-many associations

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*