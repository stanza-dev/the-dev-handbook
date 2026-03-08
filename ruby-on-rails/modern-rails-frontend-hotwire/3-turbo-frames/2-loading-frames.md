---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-lazy-loading-frames"
---

# Lazy Loading with Turbo Frames

## Introduction
Turbo Frames can load content asynchronously from a URL, either immediately or lazily when scrolled into view. This lets you serve a fast initial page and defer expensive content.

## Key Concepts
- **`src` Attribute**: A URL that Turbo fetches to populate the frame.
- **Eager Loading**: Frame fetches immediately when the page loads (default with `src`).
- **Lazy Loading**: Frame waits until visible in viewport with `loading: :lazy`.
- **Placeholder Content**: HTML inside a lazy frame shown while loading.

## Real World Context
Lazy frames are essential for pages with heavy below-the-fold content. A product page loads main details immediately but defers reviews, related products, and analytics to lazy frames, reducing initial response time.

## Deep Dive
### Eager Loading

```erb
<%= turbo_frame_tag "recent_activity",
    src: activity_feed_path do %>
  <p>Loading activity...</p>
<% end %>
```

The placeholder shows briefly while the frame makes a separate request. Turbo replaces it with the response content.

### Lazy Loading

```erb
<%= turbo_frame_tag "product_reviews",
    src: product_reviews_path(@product),
    loading: :lazy do %>
  <div class="animate-pulse">
    <div class="h-4 bg-gray-200 rounded w-3/4 mb-2"></div>
    <div class="h-4 bg-gray-200 rounded w-1/2"></div>
  </div>
<% end %>
```

The skeleton placeholder displays until the user scrolls the frame into view. Only then does Turbo fetch the content.

### Response Format

The endpoint must return a matching frame:

```erb
<!-- reviews/index.html.erb -->
<%= turbo_frame_tag "product_reviews" do %>
  <h3><%= pluralize(@reviews.count, "review") %></h3>
  <% @reviews.each do |review| %>
    <%= render review %>
  <% end %>
<% end %>
```

Turbo extracts only the matching frame. The rest of the response is discarded.

### Manual Refresh

Refresh a frame by linking to its `src`:

```erb
<%= link_to "Refresh", notifications_path,
    data: { turbo_frame: "notifications" } %>
```

## Common Pitfalls
1. **Missing frame ID in response** — The endpoint response must contain a matching `turbo_frame_tag`.
2. **Lazy loading above the fold** — Content visible on page load should use eager loading.

## Best Practices
1. **Use skeleton screens** — They provide better perceived performance than spinners.
2. **Match loading strategy to position** — Eager for visible content, lazy for below-the-fold.

## Summary
- Frames with `src` load content from a URL (eager by default, lazy with `loading: :lazy`).
- Placeholder content displays while loading.
- The server response must contain a matching frame ID.
- Frames can be manually refreshed by linking to their src URL.

## Code Examples

**A controller endpoint for a lazy-loaded frame — returns HTML with a matching turbo_frame_tag**

```ruby
# Endpoint serving lazy frame content
class ReviewsController < ApplicationController
  def index
    @product = Product.find(params[:product_id])
    @reviews = @product.reviews.includes(:author).order(created_at: :desc)
    # The view wraps content in turbo_frame_tag "product_reviews"
  end
end
```


## Resources

- [Turbo Frames Reference](https://turbo.hotwired.dev/reference/frames) — API reference for frame attributes including src and loading

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*