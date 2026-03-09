---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-low-level-caching"
---

# Low-Level Caching

## Introduction
Low-level caching lets you store any Ruby object — not just view fragments — using the `Rails.cache` API. This is essential for caching expensive database queries, API responses, and computed values.

## Key Concepts
- **Rails.cache.fetch**: The primary caching method — reads from cache if present, otherwise executes the block, stores the result, and returns it.
- **Expiration**: Time-based cache invalidation using `expires_in`.
- **Cache Keys**: Strings or arrays used to identify cached values. Arrays are joined with slashes.

## Real World Context
A dashboard that computes trending posts by joining views, likes, and comments can take 500ms+ to query. Caching the result for 15 minutes reduces load on every page view to near zero while keeping data reasonably fresh.

## Deep Dive

### Rails.cache.fetch

```ruby
# Read from cache or compute and store
posts = Rails.cache.fetch('popular_posts', expires_in: 1.hour) do
  Post.popular.limit(10).to_a
end
```

### Caching Expensive Queries

```ruby
class Post < ApplicationRecord
  def self.trending
    Rails.cache.fetch('trending_posts', expires_in: 15.minutes) do
      joins(:views)
        .where('views.created_at > ?', 24.hours.ago)
        .group(:id)
        .order('COUNT(views.id) DESC')
        .limit(10)
        .to_a  # Convert to array before caching
    end
  end
end
```

### Caching External API Calls

```ruby
class WeatherService
  def self.current(city)
    Rails.cache.fetch("weather/#{city}", expires_in: 30.minutes) do
      response = HTTP.get("https://api.weather.com/#{city}")
      JSON.parse(response.body)
    end
  end
end
```

### Cache Invalidation

```ruby
Rails.cache.delete('popular_posts')
Rails.cache.clear  # Clear entire cache
```

### Object-Based Keys

```ruby
class User < ApplicationRecord
  def dashboard_stats
    Rails.cache.fetch([self, 'dashboard_stats'], expires_in: 5.minutes) do
      { posts_count: posts.count, followers_count: followers.count }
    end
  end
end
```

## Common Pitfalls
1. **Caching ActiveRecord relations instead of arrays** — Always call `.to_a` before caching a query result. Caching the relation object stores a query, not the data.
2. **Forgetting to invalidate** — Time-based expiration works for most cases, but if data changes frequently, combine it with explicit `Rails.cache.delete` calls.

## Best Practices
1. **Use fetch, not read/write** — `fetch` combines both operations atomically and is the idiomatic Rails approach.
2. **Set reasonable expiration times** — Balance freshness and performance. Dashboard stats might tolerate 5-minute staleness; user profiles might need 1-minute expiry.

## Summary
- `Rails.cache.fetch` is the primary caching method — read or compute and store.
- Always convert query results to arrays with `.to_a` before caching.
- Use object-based cache keys like `[user, 'stats']` for automatic invalidation.
- Set `expires_in` to control staleness tolerance.

## Code Examples

**Using Rails.cache.fetch to cache an expensive trending posts query — note .to_a to convert the relation to an array**

```ruby
# Cache an expensive query for 15 minutes
trending = Rails.cache.fetch('trending_posts', expires_in: 15.minutes) do
  Post.joins(:views)
      .where('views.created_at > ?', 24.hours.ago)
      .group(:id)
      .order('COUNT(views.id) DESC')
      .limit(10)
      .to_a
end
```


## Resources

- [Low-Level Caching](https://guides.rubyonrails.org/caching_with_rails.html#low-level-caching) — Official Rails guide section on Rails.cache.fetch and low-level caching

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*