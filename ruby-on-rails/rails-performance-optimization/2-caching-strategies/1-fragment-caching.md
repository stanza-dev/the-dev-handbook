---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-fragment-caching"
---

# Fragment Caching

## Introduction
Fragment caching stores rendered HTML snippets so Rails doesn't have to re-render them on every request. It's one of the most effective ways to speed up view rendering in Rails applications.

## Key Concepts
- **Fragment Cache**: A stored HTML snippet keyed by a record's `cache_key_with_version`, which includes model name, ID, and `updated_at` timestamp.
- **Cache Digest**: Rails automatically includes a digest of the template in the cache key, so changing the template invalidates the cache.
- **Collection Caching**: Efficiently caching every item in a collection with a single multi-read from the cache store.

## Real World Context
A product listing page rendering 50 cards with prices, images, and reviews can go from 200ms to 20ms with fragment caching. Each card is cached individually, so updating one product only invalidates that card's cache.

## Deep Dive

### Basic Fragment Cache

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

The cache key is based on `post.cache_key_with_version` — cache automatically invalidates when the post is updated.

### Collection Caching

```erb
<%= render partial: 'post', collection: @posts, cached: true %>
```

Rails fetches all cache entries in one `read_multi` call!

### Touch to Invalidate Parent

```ruby
class Comment < ApplicationRecord
  belongs_to :post, touch: true
end
```

When a comment is created or updated, `post.updated_at` changes, invalidating the post's cache.

### Cache Store Configuration (Rails 8.1)

```ruby
# config/environments/production.rb

# Solid Cache (Rails 8 default — database-backed, no Redis needed)
config.cache_store = :solid_cache_store

# Redis (alternative for high-throughput caching)
config.cache_store = :redis_cache_store, {
  url: ENV['REDIS_URL'],
  expires_in: 1.day
}
```

## Common Pitfalls
1. **Forgetting touch on associations** — If a comment changes but the post's `updated_at` doesn't update, the cached post HTML will show stale comment counts.
2. **Caching user-specific content** — Include the current user in the cache key if the fragment varies per user: `cache [post, current_user]`.

## Best Practices
1. **Use collection caching** — `render partial: ..., cached: true` is far faster than individual cache calls because it uses a single multi-read operation.
2. **Use Solid Cache in production** — Rails 8's default cache store is database-backed, requires no Redis, and handles most workloads efficiently.

## Summary
- Fragment caching stores rendered HTML keyed by record version.
- Collection caching reads all entries in one multi-read call.
- Use `touch: true` on associations to auto-invalidate parent caches.
- Rails 8 defaults to Solid Cache (database-backed, no Redis needed).

## Code Examples

**Fragment caching a single record vs. collection caching with automatic multi-read**

```ruby
# In your view — cache each post card
<% cache post do %>
  <article>
    <h2><%= post.title %></h2>
    <p><%= post.body %></p>
  </article>
<% end %>

# Even faster — collection caching with multi-read
<%= render partial: 'post', collection: @posts, cached: true %>
```


## Resources

- [Caching with Rails](https://guides.rubyonrails.org/caching_with_rails.html) — Official Rails guide on caching strategies including fragment and collection caching

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*