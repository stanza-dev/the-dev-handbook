---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-lazy-loading-frames"
---

# Lazy Loading Frames

Turbo Frames can load content lazily when they become visible.

## Lazy Loading Syntax

```erb
<!-- Frame loads content when it appears in viewport -->
<%= turbo_frame_tag 'comments', 
    src: post_comments_path(@post), 
    loading: :lazy do %>
  <p>Loading comments...</p>
<% end %>
```

## Eager vs Lazy Loading

```erb
<!-- Eager: loads immediately (default) -->
<%= turbo_frame_tag 'stats', src: stats_path %>

<!-- Lazy: loads when scrolled into view -->
<%= turbo_frame_tag 'stats', src: stats_path, loading: :lazy %>
```

## Controller Response for Frames

```ruby
# posts_controller.rb
def comments
  @post = Post.find(params[:post_id])
  @comments = @post.comments.includes(:author)
  
  # Render just the frame content
  render partial: 'comments/list', locals: { comments: @comments }
end
```

```erb
<!-- comments/_list.html.erb -->
<%= turbo_frame_tag 'comments' do %>
  <% comments.each do |comment| %>
    <%= render comment %>
  <% end %>
<% end %>
```

## Refreshing Frame Content

```erb
<%= turbo_frame_tag 'notifications', src: notifications_path do %>
  <%= render @notifications %>
<% end %>

<!-- Refresh button -->
<%= link_to 'Refresh', notifications_path, 
    data: { turbo_frame: 'notifications' } %>
```

## Auto-Refreshing

```erb
<!-- Refresh every 30 seconds -->
<%= turbo_frame_tag 'live_stats', 
    src: stats_path,
    data: { controller: 'auto-refresh', 
            auto_refresh_interval_value: 30000 } do %>
  Loading...
<% end %>
```

```javascript
// controllers/auto_refresh_controller.js
import { Controller } from '@hotwired/stimulus'

export default class extends Controller {
  static values = { interval: Number }
  
  connect() {
    this.startRefreshing()
  }
  
  disconnect() {
    this.stopRefreshing()
  }
  
  startRefreshing() {
    this.refreshTimer = setInterval(() => {
      this.element.reload()
    }, this.intervalValue)
  }
  
  stopRefreshing() {
    clearInterval(this.refreshTimer)
  }
}
```

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*