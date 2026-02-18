---
source_course: "rails-foundations"
source_lesson: "rails-foundations-has-many-through"
---

# Many-to-Many with has_many :through

When two models need to relate to each other in both directions, you need a many-to-many relationship. Rails handles this with `has_many :through`.

## The Problem

Consider articles and tags:
- An article can have many tags
- A tag can belong to many articles

You need a **join table** to connect them.

## Setting Up has_many :through

### The Models

```ruby
# app/models/article.rb
class Article < ApplicationRecord
  has_many :article_tags
  has_many :tags, through: :article_tags
end

# app/models/tag.rb
class Tag < ApplicationRecord
  has_many :article_tags
  has_many :articles, through: :article_tags
end

# app/models/article_tag.rb (the join model)
class ArticleTag < ApplicationRecord
  belongs_to :article
  belongs_to :tag
end
```

### The Migrations

```ruby
# Create tags table
class CreateTags < ActiveRecord::Migration[8.0]
  def change
    create_table :tags do |t|
      t.string :name
      t.timestamps
    end
  end
end

# Create join table
class CreateArticleTags < ActiveRecord::Migration[8.0]
  def change
    create_table :article_tags do |t|
      t.references :article, null: false, foreign_key: true
      t.references :tag, null: false, foreign_key: true

      t.timestamps
    end

    # Prevent duplicate associations
    add_index :article_tags, [:article_id, :tag_id], unique: true
  end
end
```

### Using the Association

```ruby
# Create some tags
ruby_tag = Tag.create(name: "Ruby")
rails_tag = Tag.create(name: "Rails")

# Create an article with tags
article = Article.create(title: "Learning Rails")
article.tags << ruby_tag
article.tags << rails_tag

# Or assign all at once
article.tags = [ruby_tag, rails_tag]

# Query both directions
article.tags          # => [ruby_tag, rails_tag]
ruby_tag.articles     # => [article]

# Add tags by ID
article.tag_ids = [1, 2, 3]
```

## Adding Attributes to the Join

The join model can have its own attributes:

```ruby
class ArticleTag < ApplicationRecord
  belongs_to :article
  belongs_to :tag

  # Additional column: position
end

# Create with extra attributes
article.article_tags.create(tag: ruby_tag, position: 1)
```

## has_and_belongs_to_many (HABTM)

A simpler but less flexible alternative:

```ruby
class Article < ApplicationRecord
  has_and_belongs_to_many :tags
end

class Tag < ApplicationRecord
  has_and_belongs_to_many :articles
end
```

**Use `has_many :through` instead** when you need:
- Additional attributes on the join
- Validations on the join
- Callbacks on the join
- To query the join model directly

## Practical Example: Forms with Checkboxes

```erb
<%= form_with model: @article do |f| %>
  <% Tag.all.each do |tag| %>
    <label>
      <%= check_box_tag "article[tag_ids][]", tag.id, @article.tag_ids.include?(tag.id) %>
      <%= tag.name %>
    </label>
  <% end %>
<% end %>
```

```ruby
# In controller
def article_params
  params.require(:article).permit(:title, :body, tag_ids: [])
end
```

## Resources

- [Active Record Associations - has_many :through](https://guides.rubyonrails.org/association_basics.html#the-has-many-through-association) — Guide to many-to-many associations

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*