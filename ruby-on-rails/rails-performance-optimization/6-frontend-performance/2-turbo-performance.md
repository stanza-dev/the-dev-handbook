---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-turbo-performance"
---

# Turbo Performance Patterns

## Introduction
Turbo makes Rails applications feel as fast as single-page apps by avoiding full page reloads. It intercepts link clicks and form submissions, replacing only the parts of the page that changed.

## Key Concepts
- **Turbo Drive**: Automatically intercepts all navigation and uses AJAX to swap only the `<body>`, preserving the `<head>` and avoiding full page reloads.
- **Turbo Frames**: Scoped regions of a page that update independently — clicking a link inside a frame only updates that frame.
- **Turbo Streams**: Server-sent HTML fragments that append, prepend, replace, or remove elements on the page in real time.

## Real World Context
A product listing with pagination, filters, and search can feel instant with Turbo Frames: clicking page 2 only replaces the product grid, not the header, sidebar, or footer. Users perceive near-zero load times.

## Deep Dive

### Turbo Drive (Default)

```erb
<!-- Regular link — handled by Turbo automatically -->
<%= link_to 'Products', products_path %>

<!-- Disable Turbo for specific link -->
<%= link_to 'Download', file_path, data: { turbo: false } %>
```

### Turbo Frames for Partial Updates

```erb
<!-- Only this frame updates on navigation -->
<%= turbo_frame_tag 'product_list' do %>
  <%= render @products %>
  <%= link_to 'Next', products_path(page: 2) %>
<% end %>

<!-- Lazy load content -->
<%= turbo_frame_tag 'comments', src: post_comments_path(@post), loading: :lazy do %>
  <p>Loading comments...</p>
<% end %>
```

### Turbo Streams for Live Updates

```ruby
def create
  @comment = @post.comments.create!(comment_params)
  respond_to do |format|
    format.turbo_stream
    format.html { redirect_to @post }
  end
end
```

```erb
<!-- app/views/comments/create.turbo_stream.erb -->
<%= turbo_stream.append 'comments', @comment %>
<%= turbo_stream.update 'comment_count', @post.comments.count %>
```

### Prefetching on Hover

```erb
<%= link_to 'Products', products_path, data: { turbo_prefetch: true } %>
```

## Common Pitfalls
1. **Breaking out of frames accidentally** — Links inside a Turbo Frame stay within the frame by default. To navigate the full page, add `data-turbo-frame="_top"`.
2. **Not providing fallback for non-Turbo requests** — Always handle both `format.turbo_stream` and `format.html` in your respond_to block.

## Best Practices
1. **Wrap pagination in Turbo Frames** — Users see instant page transitions without full reloads.
2. **Use lazy-loading frames for below-the-fold content** — Defer loading comments, related posts, or sidebars until the user needs them.

## Summary
- Turbo Drive intercepts all navigation automatically — no code needed.
- Turbo Frames scope updates to specific regions of the page.
- Turbo Streams enable real-time append/replace/remove operations.
- Use `loading: :lazy` on Turbo Frames to defer below-the-fold content.

## Code Examples

**Lazy-loading a Turbo Frame — the reviews section loads asynchronously, keeping initial page load fast**

```ruby
# Lazy-loaded Turbo Frame — content loads only when visible
<%= turbo_frame_tag 'product_reviews',
      src: product_reviews_path(@product),
      loading: :lazy do %>
  <p>Loading reviews...</p>
<% end %>

# Server responds with just the frame content
# No full page render needed — only the reviews HTML
```


## Resources

- [Turbo Handbook](https://turbo.hotwired.dev/handbook/introduction) — Official Hotwire Turbo documentation covering Drive, Frames, and Streams

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*