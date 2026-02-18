---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-broadcasting-streams"
---

# Broadcasting Streams

Turbo Streams can broadcast updates over WebSockets using Action Cable.

## Setting Up Broadcasting

```ruby
# app/models/comment.rb
class Comment < ApplicationRecord
  belongs_to :post
  
  # Broadcast after create
  after_create_commit -> {
    broadcast_append_to post,
      target: 'comments',
      partial: 'comments/comment'
  }
  
  # Broadcast after update
  after_update_commit -> {
    broadcast_replace_to post
  }
  
  # Broadcast after destroy
  after_destroy_commit -> {
    broadcast_remove_to post
  }
end
```

## Subscribing to Streams

```erb
<!-- Subscribe to updates for this post -->
<%= turbo_stream_from @post %>

<div id="comments">
  <%= render @post.comments %>
</div>
```

## Simplified Callbacks

```ruby
class Message < ApplicationRecord
  belongs_to :conversation
  
  # Broadcasts append, replace, and remove automatically
  broadcasts_to :conversation
end
```

## Custom Broadcast Targets

```ruby
class Notification < ApplicationRecord
  belongs_to :user
  
  after_create_commit do
    broadcast_prepend_to [
      user, 'notifications'
    ],
    target: 'notifications_list'
  end
end
```

```erb
<!-- User's notification sidebar -->
<%= turbo_stream_from current_user, 'notifications' %>

<div id="notifications_list">
  <%= render current_user.notifications.unread %>
</div>
```

## Manual Broadcasting

```ruby
class OrdersController < ApplicationController
  def update_status
    @order.update!(status: params[:status])
    
    # Broadcast to admin dashboard
    Turbo::StreamsChannel.broadcast_replace_to(
      'admin_orders',
      target: dom_id(@order),
      partial: 'admin/orders/order',
      locals: { order: @order }
    )
    
    # Broadcast to customer
    Turbo::StreamsChannel.broadcast_update_to(
      @order.customer,
      target: 'order_status',
      html: @order.status.humanize
    )
  end
end
```

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*