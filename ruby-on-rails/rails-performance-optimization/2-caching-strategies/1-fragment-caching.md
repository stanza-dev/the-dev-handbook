---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-fragment-caching"
---

# Fragment Caching

Fragment caching stores rendered HTML snippets to avoid re-rendering.

## Basic Fragment Cache

```erb
<% @posts.each do |post| %>
  <% cache post do %>
    <article>
      <h2><%= post.title %></h2>
      <p><%= post.body %></p>
      <span>By <%= post.author.name %></span>
    </article>
  <% end %>
<% end %>
```

The cache key is based on `post.cache_key_with_version`, which includes:
- Model name
- Record ID
- `updated_at` timestamp

Cache automatically invalidates when the post is updated!

## Cache Keys

```erb
<!-- Simple key -->
<% cache 'sidebar' do %>
  <%= render 'sidebar' %>
<% end %>

<!-- Key with object -->
<% cache @post do %>
  ...
<% end %>

<!-- Array key (for dependent objects) -->
<% cache [@post, current_user] do %>
  ...
<% end %>

<!-- With explicit version -->
<% cache [@post, 'v2'] do %>
  ...
<% end %>
```

## Collection Caching

Efficiently cache collections:

```erb
<%= render partial: 'post', collection: @posts, cached: true %>
```

Rails fetches all cache entries in one call!

## Touching Associations

Invalidate parent cache when children change:

```ruby
class Comment < ApplicationRecord
  belongs_to :post, touch: true
end
```

Now when a comment is created/updated, `post.updated_at` changes, invalidating the cache.

## Cache Store Configuration

```ruby
# config/environments/production.rb

# Memory store (default, not shared between processes)
config.cache_store = :memory_store, { size: 64.megabytes }

# Redis (recommended for production)
config.cache_store = :redis_cache_store, {
  url: ENV['REDIS_URL'],
  expires_in: 1.day
}

# Memcached
config.cache_store = :mem_cache_store, 'cache.example.com'
```

See [Caching Guide](https://guides.rubyonrails.org/caching_with_rails.html).

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*