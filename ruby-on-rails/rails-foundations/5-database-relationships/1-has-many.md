---
source_course: "rails-foundations"
source_lesson: "rails-foundations-belongs-to-has-many"
---

# belongs_to and has_many Associations

## Introduction

Associations connect your models together, representing relationships between database tables. The one-to-many relationship is the most common pattern in web applications, and Rails makes it remarkably easy to set up.

## Key Concepts

- **`has_many`**: Declares that a model has zero or more instances of another model (e.g., an article has many comments).
- **`belongs_to`**: Declares that a model references another model via a foreign key (e.g., a comment belongs to an article).
- **Foreign key**: The column (e.g., `article_id`) on the `belongs_to` side that stores the reference to the parent record.
- **`dependent: :destroy`**: An option that automatically deletes associated records when the parent is deleted.

## Real World Context

Almost every database has one-to-many relationships: users have many posts, orders have many items, categories have many products. Understanding associations is essential for modeling any real-world domain in Rails.

## Deep Dive

### Setting Up the Models

```ruby
class Article < ApplicationRecord
  has_many :comments, dependent: :destroy
end

class Comment < ApplicationRecord
  belongs_to :article
end
```

### The Migration

```ruby
class CreateComments < ActiveRecord::Migration[8.1]
  def change
    create_table :comments do |t|
      t.text :body
      t.references :article, null: false, foreign_key: true
      t.timestamps
    end
  end
end
```

`t.references :article` creates an `article_id` column, an index, and a foreign key constraint.

### Using Associations

```ruby
article = Article.create(title: "Rails Associations")
comment = article.comments.create(body: "Great article!")

article.comments          # All comments for this article
comment.article           # The article this comment belongs to
article.comments.count    # Number of comments
article.comments.build(body: "Draft")  # Build without saving
```

### Association Methods

`has_many :comments` generates:

```ruby
article.comments              # Collection
article.comments << comment   # Add to collection
article.comments.delete(comment)  # Remove
article.comments.destroy_all  # Delete all
article.comments.find(id)     # Find specific
article.comments.where(...)   # Query
article.comments.build(...)   # Build new
article.comments.create(...)  # Create new
```

### Dependent Option

```ruby
has_many :comments, dependent: :destroy     # Delete with callbacks
has_many :comments, dependent: :delete_all  # Delete without callbacks
has_many :comments, dependent: :nullify     # Set FK to NULL
has_many :comments, dependent: :restrict_with_error  # Prevent deletion
```

### Optional Associations

```ruby
class Comment < ApplicationRecord
  belongs_to :article, optional: true  # article_id can be NULL
end
```

## Common Pitfalls

- **Forgetting the foreign key migration**: The `belongs_to` side needs a foreign key column. Always use `t.references` in migrations.
- **Not setting `dependent`**: Without it, deleting a parent leaves orphaned child records in the database.
- **Confusing which side gets `belongs_to`**: The model with the foreign key column always uses `belongs_to`.

## Best Practices

- Always set `dependent:` on `has_many` associations to prevent orphaned records.
- Use `t.references` with `foreign_key: true` to enforce referential integrity at the database level.
- Use `null: false` on foreign key columns when the relationship is required.

## Summary

- `has_many` and `belongs_to` set up one-to-many relationships between models.
- The `belongs_to` side holds the foreign key column (e.g., `article_id`).
- `t.references :article, foreign_key: true` creates the column, index, and constraint.
- Use `dependent: :destroy` to automatically clean up child records.
- Rails generates many helpful methods on both sides of the association.

## Code Examples

**A one-to-many association where an article has many comments. The Comment model holds the foreign key (article_id), and dependent: :destroy cleans up comments when an article is deleted.**

```ruby
class Article < ApplicationRecord
  has_many :comments, dependent: :destroy
end

class Comment < ApplicationRecord
  belongs_to :article
end

# Usage
article = Article.create(title: "Rails")
article.comments.create(body: "Great!")
article.comments.count  # => 1
```


## Resources

- [Active Record Associations](https://guides.rubyonrails.org/association_basics.html) — Complete guide to Rails associations

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*