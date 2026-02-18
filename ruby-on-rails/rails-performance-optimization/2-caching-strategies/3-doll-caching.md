---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-russian-doll-caching"
---

# Russian Doll Caching

Russian doll caching nests cached fragments inside each other.

## The Pattern

```erb
<!-- Outer cache: entire post list -->
<% cache ['posts', @posts.maximum(:updated_at)] do %>
  <% @posts.each do |post| %>
    
    <!-- Middle cache: individual post -->
    <% cache post do %>
      <article>
        <h2><%= post.title %></h2>
        
        <!-- Inner cache: comments -->
        <% cache [post, 'comments', post.comments.maximum(:updated_at)] do %>
          <div class="comments">
            <% post.comments.each do |comment| %>
              
              <!-- Innermost cache: single comment -->
              <% cache comment do %>
                <p><%= comment.body %></p>
              <% end %>
              
            <% end %>
          </div>
        <% end %>
        
      </article>
    <% end %>
    
  <% end %>
<% end %>
```

## How It Works

1. When a comment changes, only that comment's cache invalidates
2. The comments container cache invalidates (new `maximum(:updated_at)`)
3. The post cache invalidates (`touch: true` on comment)
4. The posts list cache invalidates (new `maximum(:updated_at)`)

But cached siblings stay cached!

## Setting Up Touch

```ruby
class Comment < ApplicationRecord
  belongs_to :post, touch: true
end

class Post < ApplicationRecord
  belongs_to :author, touch: true
  has_many :comments, dependent: :destroy
end
```

## Cache Dependencies

```erb
<!-- Include all dependencies in key -->
<% cache [post, post.author, post.comments.maximum(:updated_at)] do %>
  <h2><%= post.title %></h2>
  <p>By <%= post.author.name %></p>
  <span><%= post.comments.count %> comments</span>
<% end %>
```

## Digest-Based Keys

Rails automatically includes template digest in cache keys:

```erb
<!-- If template changes, cache automatically invalidates -->
<% cache post do %>
  <%= post.title %>  <!-- Change this, cache refreshes -->
<% end %>
```

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*