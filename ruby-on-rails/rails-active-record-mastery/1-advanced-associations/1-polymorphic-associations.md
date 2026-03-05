---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-polymorphic-associations"
---

# Polymorphic Associations

## Introduction
Polymorphic associations allow a model to belong to multiple other models on a single association. If you have ever needed comments on articles, photos, and videos without duplicating tables, polymorphic associations are the answer.

## Key Concepts
- **Polymorphic association**: A single association that can point to records in different tables, identified by a `_type` and `_id` column pair.
- **`commentable`**: A conventional naming pattern for the polymorphic interface (the "able" suffix signals polymorphism).
- **`as:` option**: Used on the `has_many` side to declare which polymorphic interface a model participates in.

## Real World Context
Almost every production Rails application has at least one polymorphic association. Comments, attachments, tags, likes, and activity feeds are classic examples. Without polymorphism, you would need a separate join table or model for every combination, leading to massive code duplication.

## Deep Dive

### The Problem

Imagine you want comments on multiple types of content. Without polymorphism, you would need separate models:

```ruby
# Without polymorphism, you'd need:
class ArticleComment < ApplicationRecord
  belongs_to :article
end

class PhotoComment < ApplicationRecord
  belongs_to :photo
end

class VideoComment < ApplicationRecord
  belongs_to :video
end
```

This leads to code duplication and separate tables for each type.

### The Polymorphic Solution

With polymorphism, one Comment model works for all:

```ruby
# app/models/comment.rb
class Comment < ApplicationRecord
  belongs_to :commentable, polymorphic: true
end

# app/models/article.rb
class Article < ApplicationRecord
  has_many :comments, as: :commentable
end

# app/models/photo.rb
class Photo < ApplicationRecord
  has_many :comments, as: :commentable
end

# app/models/video.rb
class Video < ApplicationRecord
  has_many :comments, as: :commentable
end
```

The `as: :commentable` option tells Rails that this model participates in the `commentable` polymorphic interface.

### The Migration

Polymorphic associations need two columns, `_type` and `_id`:

```ruby
class CreateComments < ActiveRecord::Migration[8.1]
  def change
    create_table :comments do |t|
      t.text :body
      t.references :commentable, polymorphic: true, null: false
      # Creates: commentable_id (integer) and commentable_type (string)
      # Plus an index on both columns

      t.timestamps
    end
  end
end
```

The `commentable_type` stores the class name ("Article", "Photo", etc.) and `commentable_id` stores the primary key of that record.

### Using Polymorphic Associations

```ruby
# Create comments on different types
article = Article.create(title: "Rails Guide")
photo = Photo.create(url: "sunset.jpg")

article.comments.create(body: "Great article!")
photo.comments.create(body: "Beautiful photo!")

# Access comments
article.comments  # => [#<Comment body: "Great article!"...>]
photo.comments    # => [#<Comment body: "Beautiful photo!"...>]

# Access parent from comment
comment = Comment.first
comment.commentable        # => #<Article title: "Rails Guide"...>
comment.commentable_type   # => "Article"
comment.commentable_id     # => 1
```

### Querying Polymorphic Associations

```ruby
# Find all comments on articles
Comment.where(commentable_type: "Article")

# Find comments for a specific article
Comment.where(commentable: article)

# Eager load the parent (be careful - multiple queries)
Comment.includes(:commentable).each do |comment|
  puts comment.commentable.title rescue comment.commentable.url
end
```

Note that eager loading polymorphic associations issues one query per type, which can be expensive if you have many types.

### Rails 8.1 Deprecation Support

Rails 8.1 introduces the `deprecated: true` option for associations, which logs a deprecation warning whenever the association is accessed:

```ruby
class Author < ApplicationRecord
  has_many :legacy_posts, deprecated: true
end
```

This is useful when migrating away from a polymorphic pattern or renaming associations.

## Common Pitfalls
1. **Cannot use foreign key constraints** -- Because the `_type` column means the `_id` can reference different tables, the database cannot enforce referential integrity. Use application-level validations instead.
2. **N+1 queries on eager loading** -- `includes(:commentable)` issues separate queries per type. For performance-critical pages, consider a denormalized cache column or a different design.
3. **Orphaned records** -- Without foreign keys, deleting a parent does not cascade. Always use `dependent: :destroy` on the `has_many` side.

## Best Practices
1. **Index the polymorphic columns** -- Always add a composite index on `[commentable_type, commentable_id]` for query performance.
2. **Use `dependent: :destroy`** -- Since you cannot rely on database-level cascading, explicitly declare cleanup behavior on the parent model.
3. **Consider delegated types for complex cases** -- If your polymorphic types have very different attributes, Rails 6.1+ delegated types may be a better fit.

## Summary
- Polymorphic associations let one model belong to many different models via `_type` and `_id` columns.
- Use `belongs_to :x, polymorphic: true` and `has_many :items, as: :x`.
- You cannot use database foreign key constraints with polymorphic associations.
- Eager loading issues one query per type, so be mindful of performance.
- Common uses include comments, attachments, tags, likes, and activity feeds.

## Code Examples

**Defining and using a polymorphic association -- Comment can belong to any model that declares `has_many :comments, as: :commentable`**

```ruby
# app/models/comment.rb
class Comment < ApplicationRecord
  belongs_to :commentable, polymorphic: true
end

# app/models/article.rb
class Article < ApplicationRecord
  has_many :comments, as: :commentable, dependent: :destroy
end

# Usage
article = Article.create(title: "Rails Guide")
article.comments.create(body: "Great article!")

comment = Comment.first
comment.commentable       # => #<Article ...>
comment.commentable_type  # => "Article"
```


## Resources

- [Active Record Associations - Polymorphic](https://guides.rubyonrails.org/association_basics.html#polymorphic-associations) — Official guide to polymorphic associations

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*