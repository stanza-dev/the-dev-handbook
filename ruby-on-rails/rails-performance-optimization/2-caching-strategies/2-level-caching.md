---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-low-level-caching"
---

# Low Level Caching

Low-level caching stores any Ruby object, not just view fragments.

## Rails.cache API

```ruby
# Write to cache
Rails.cache.write('popular_posts', @posts, expires_in: 1.hour)

# Read from cache
posts = Rails.cache.read('popular_posts')

# Fetch: read or compute and write
posts = Rails.cache.fetch('popular_posts', expires_in: 1.hour) do
  Post.popular.limit(10).to_a
end
```

## Caching Expensive Queries

```ruby
class Post < ApplicationRecord
  def self.trending
    Rails.cache.fetch('trending_posts', expires_in: 15.minutes) do
      joins(:views)
        .where('views.created_at > ?', 24.hours.ago)
        .group(:id)
        .order('COUNT(views.id) DESC')
        .limit(10)
        .to_a  # Important: convert to array
    end
  end
end
```

## Caching API Calls

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

## Cache Keys with Objects

```ruby
class User < ApplicationRecord
  def dashboard_stats
    Rails.cache.fetch([self, 'dashboard_stats'], expires_in: 5.minutes) do
      {
        posts_count: posts.count,
        comments_count: comments.count,
        followers_count: followers.count,
        computed_at: Time.current
      }
    end
  end
end
```

## Conditional Caching

```ruby
def expensive_calculation
  if should_cache?
    Rails.cache.fetch(cache_key, expires_in: 1.hour) do
      perform_calculation
    end
  else
    perform_calculation
  end
end
```

## Cache Invalidation

```ruby
# Delete specific key
Rails.cache.delete('popular_posts')

# Delete matching pattern (Redis only)
Rails.cache.delete_matched('posts/*')

# Clear entire cache
Rails.cache.clear
```

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*