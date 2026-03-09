---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-russian-doll-caching"
---

# Russian Doll Caching

## Introduction
Russian doll caching nests cached fragments inside each other, so when an inner fragment changes, only that fragment and its direct parents are invalidated — siblings remain cached. This dramatically reduces cache misses.

## Key Concepts
- **Nested Caching**: Cache fragments contain other cache fragments, forming a hierarchy.
- **Touch Propagation**: When a child record updates, `touch: true` propagates the timestamp change up the chain.
- **Cache Digest**: Rails includes a template digest in cache keys, so template changes automatically invalidate caches.

## Real World Context
An e-commerce category page with 100 products, each showing reviews: when one review is added, only that product's cache invalidates. The other 99 products serve from cache instantly.

## Deep Dive

### The Pattern

```erb
<% cache ['posts', @posts.maximum(:updated_at)] do %>
  <% @posts.each do |post| %>
    <% cache post do %>
      <article>
        <h2><%= post.title %></h2>
        <% cache [post, 'comments'] do %>
          <div class="comments">
            <% post.comments.each do |comment| %>
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

### Setting Up Touch Propagation

```ruby
class Comment < ApplicationRecord
  belongs_to :post, touch: true
end

class Post < ApplicationRecord
  belongs_to :author, touch: true
  has_many :comments, dependent: :destroy
end
```

When a comment changes:
1. Comment's cache invalidates
2. Post's `updated_at` updates via `touch: true`
3. Post's cache invalidates
4. Sibling posts remain cached

### Template Digest Keys

Rails automatically includes a digest of the template in cache keys:

```erb
<% cache post do %>
  <%= post.title %>  <!-- Change this template, cache auto-refreshes -->
<% end %>
```

## Common Pitfalls
1. **Missing touch declarations** — If you forget `touch: true` on the child association, parent caches won't invalidate when children change.
2. **Too many nesting levels** — More than 3-4 levels of nesting adds complexity without much benefit. Keep it practical.

## Best Practices
1. **Always add touch: true on belongs_to** — Any association whose parent is cached should propagate timestamp changes.
2. **Use maximum(:updated_at) for collection keys** — This ensures the outer cache invalidates when any item in the collection changes.

## Summary
- Russian doll caching nests fragments so siblings stay cached when one changes.
- `touch: true` propagates timestamp changes up the association chain.
- Rails auto-includes template digests in cache keys.
- Keep nesting to 3-4 levels maximum for maintainability.

## Code Examples

**Setting up touch propagation — changes bubble up from comment to post to category, invalidating caches at each level**

```ruby
# Touch propagation setup
class Comment < ApplicationRecord
  belongs_to :post, touch: true
end

class Post < ApplicationRecord
  belongs_to :category, touch: true
  has_many :comments
end

# When a comment is saved:
# 1. comment.updated_at changes
# 2. post.updated_at changes (touch)
# 3. category.updated_at changes (touch)
# All related caches invalidate automatically
```


## Resources

- [Russian Doll Caching](https://guides.rubyonrails.org/caching_with_rails.html#russian-doll-caching) — Official Rails guide on nested fragment caching with touch propagation

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*