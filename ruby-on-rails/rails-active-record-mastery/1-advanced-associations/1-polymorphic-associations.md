---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-polymorphic-associations"
---

# Polymorphic Associations

Polymorphic associations allow a model to belong to multiple other models on a single association. This is perfect for features like comments that can belong to articles, photos, or videos.

## The Problem

Imagine you want comments on multiple types of content:

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

This leads to code duplication and separate tables for each.

## The Polymorphic Solution

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

## The Migration

Polymorphic associations need two columns: `_type` and `_id`:

```ruby
class CreateComments < ActiveRecord::Migration[8.0]
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

The `commentable_type` stores the class name ("Article", "Photo", etc.).

## Using Polymorphic Associations

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

## Common Use Cases

- **Comments**: On posts, photos, videos, products
- **Attachments**: Files attached to messages, tasks, projects
- **Tags/Taggings**: Tags on articles, products, users
- **Likes/Favorites**: Users liking various content types
- **Activity Feeds**: Tracking actions on different models

## Querying Polymorphic Associations

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

## Limitations

- Cannot use foreign key constraints (type varies)
- Eager loading loads each type separately
- Need to handle each type differently in views

## Resources

- [Active Record Associations - Polymorphic](https://guides.rubyonrails.org/association_basics.html#polymorphic-associations) — Official guide to polymorphic associations

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*