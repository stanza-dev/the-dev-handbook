---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-connection-management"
---

# Connection Management and Performance

Managing thousands of WebSocket connections requires careful attention to resources.

## Connection Limits

```ruby
# config/initializers/action_cable.rb
Rails.application.config.action_cable.connection_class = -> { ApplicationCable::Connection }

# Limit connections per user
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :current_user

    def connect
      self.current_user = find_verified_user
      limit_connections
    end

    private

    def limit_connections
      # Limit to 5 connections per user
      existing = ActionCable.server.connections.count do |conn|
        conn.current_user&.id == current_user.id
      end

      if existing >= 5
        reject_unauthorized_connection
      end
    end
  end
end
```

## Disconnect Stale Connections

```ruby
# config/initializers/action_cable.rb
Rails.application.config.action_cable.disconnect_delay = 3.seconds

# In connection.rb
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    def connect
      # Reject inactive users
      reject_unauthorized_connection if current_user.suspended?
    end
  end
end
```

## Memory Optimization

```ruby
# Don't store large objects in connection
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :user_id  # Just the ID, not the whole object

    def current_user
      @current_user ||= User.find_by(id: user_id)
    end
  end
end
```

## Channel Authorization

```ruby
class ChatChannel < ApplicationCable::Channel
  def subscribed
    room = Room.find_by(id: params[:room_id])

    if room.nil?
      reject
    elsif !room.member?(current_user)
      reject
    else
      stream_for room
    end
  end
end
```

## Rate Limiting Broadcasts

```ruby
class Comment < ApplicationRecord
  after_create_commit :broadcast_comment

  private

  def broadcast_comment
    # Throttle broadcasts
    return if recently_broadcast?

    ArticleChannel.broadcast_to(article, { ... })
    touch_broadcast_time
  end

  def recently_broadcast?
    Rails.cache.exist?(broadcast_cache_key)
  end

  def touch_broadcast_time
    Rails.cache.write(broadcast_cache_key, true, expires_in: 1.second)
  end

  def broadcast_cache_key
    "broadcast:article:#{article_id}"
  end
end
```

## Worker Pool Configuration

```ruby
# config/environments/production.rb
config.action_cable.worker_pool_size = ENV.fetch('CABLE_WORKER_POOL') { 4 }
config.action_cable.max_connections = ENV.fetch('CABLE_MAX_CONNECTIONS') { 1000 }
```

## Monitoring Connections

```ruby
# lib/tasks/cable.rake
namespace :cable do
  desc "Show connection statistics"
  task stats: :environment do
    puts "Total connections: #{ActionCable.server.connections.count}"

    user_counts = ActionCable.server.connections.group_by { |c|
      c.current_user&.id
    }.transform_values(&:count)

    puts "Connections by user: #{user_counts}"
  end
end
```

## Resources

- [Action Cable Performance](https://guides.rubyonrails.org/action_cable_overview.html#configuration) — Performance tuning for Action Cable

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*