---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-turbo-performance"
---

# Turbo Performance

Turbo makes applications feel faster by avoiding full page reloads.

## Turbo Drive (Default)

Automatically converts link clicks and form submissions to AJAX:

```erb
<!-- Regular link - handled by Turbo automatically -->
<%= link_to 'Products', products_path %>

<!-- Disable Turbo for specific link -->
<%= link_to 'Download', file_path, data: { turbo: false } %>
```

## Turbo Frames for Partial Updates

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

## Turbo Streams for Live Updates

```ruby
# app/controllers/comments_controller.rb
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
<%= turbo_stream.replace 'new_comment', partial: 'comments/form', locals: { comment: Comment.new } %>
```

## Prefetching

```erb
<!-- Prefetch on hover -->
<%= link_to 'Products', products_path, data: { turbo_prefetch: true } %>
```

```javascript
// Programmatic prefetch
import { Turbo } from '@hotwired/turbo-rails'

document.querySelectorAll('.prefetch-link').forEach(link => {
  link.addEventListener('mouseenter', () => {
    Turbo.cache.fetch(link.href)
  })
})
```

## Progress Indicator

```css
.turbo-progress-bar {
  background-color: #3b82f6;
  height: 3px;
}
```

```javascript
// Show custom loading state
addEventListener('turbo:before-fetch-request', () => {
  document.body.classList.add('loading')
})

addEventListener('turbo:before-fetch-response', () => {
  document.body.classList.remove('loading')
})
```

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*