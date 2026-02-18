---
source_course: "rails-foundations"
source_lesson: "rails-foundations-belongs-to-has-many"
---

# belongs_to and has_many Associations

Associations connect your models together, representing relationships between database tables. The most common is the one-to-many relationship.

## One-to-Many Relationships

Consider articles and comments: one article has many comments, each comment belongs to one article.

### Setting Up the Models

```ruby
# app/models/article.rb
class Article < ApplicationRecord
  has_many :comments
end

# app/models/comment.rb
class Comment < ApplicationRecord
  belongs_to :article
end
```

### The Migration

The `belongs_to` side needs a foreign key:

```ruby
# db/migrate/XXXXX_create_comments.rb
class CreateComments < ActiveRecord::Migration[8.0]
  def change
    create_table :comments do |t|
      t.text :body
      t.references :article, null: false, foreign_key: true

      t.timestamps
    end
  end
end
```

`t.references :article` creates:
- An `article_id` column
- An index on that column
- A foreign key constraint (with `foreign_key: true`)

### Using Associations

```ruby
# Create an article
article = Article.create(title: "Rails Associations")

# Add comments to an article
comment = article.comments.create(body: "Great article!")
# OR
comment = Comment.create(body: "Thanks!", article: article)

# Access the relationship
article.comments          # => All comments for this article
comment.article           # => The article this comment belongs to

# Build (create without saving)
new_comment = article.comments.build(body: "Draft comment")
new_comment.save

# Check associations
article.comments.count    # => 2
article.comments.empty?   # => false
comment.article.title     # => "Rails Associations"
```

## Association Methods

When you declare `has_many :comments`, Rails generates:

```ruby
article.comments              # Collection of comments
article.comments << comment   # Add to collection
article.comments.delete(comment)  # Remove from collection
article.comments.destroy_all  # Delete all comments
article.comments.size         # Count comments
article.comments.find(id)     # Find specific comment
article.comments.where(...)   # Query comments
article.comments.build(...)   # Build new comment
article.comments.create(...)  # Create new comment
article.comment_ids           # Array of comment IDs
article.comment_ids = [1,2,3] # Set comments by IDs
```

## Dependent Option

What happens to comments when an article is deleted?

```ruby
class Article < ApplicationRecord
  # Delete all comments when article is deleted
  has_many :comments, dependent: :destroy

  # Other options:
  # dependent: :delete_all  # Faster, skips callbacks
  # dependent: :nullify     # Set article_id to NULL
  # dependent: :restrict_with_error  # Prevent deletion if comments exist
end
```

## Optional Associations

By default, `belongs_to` requires the parent to exist. Make it optional:

```ruby
class Comment < ApplicationRecord
  belongs_to :article, optional: true  # article_id can be NULL
end
```

## Resources

- [Active Record Associations](https://guides.rubyonrails.org/association_basics.html) — Complete guide to Rails associations

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*