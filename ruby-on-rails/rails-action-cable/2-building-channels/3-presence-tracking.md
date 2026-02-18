---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-presence-tracking"
---

# Presence Tracking

Track which users are online and show presence indicators.

## Presence Channel

```ruby
# app/channels/presence_channel.rb
class PresenceChannel < ApplicationCable::Channel
  def subscribed
    stream_from "presence"
    add_user_to_presence
    broadcast_presence
  end

  def unsubscribed
    remove_user_from_presence
    broadcast_presence
  end

  private

  def add_user_to_presence
    presence_list << current_user.id
  end

  def remove_user_from_presence
    presence_list.delete(current_user.id)
  end

  def presence_list
    @presence_list ||= Rails.cache.fetch("presence_users") { Set.new }
  end

  def broadcast_presence
    users = User.where(id: presence_list.to_a).pluck(:id, :name)

    ActionCable.server.broadcast("presence", {
      type: "presence_update",
      users: users.map { |id, name| { id: id, name: name } },
      count: users.size
    })
  end
end
```

## Using Redis for Presence

More reliable with multiple servers:

```ruby
# app/channels/presence_channel.rb
class PresenceChannel < ApplicationCable::Channel
  def subscribed
    stream_from "presence"
    add_user_presence
    broadcast_presence_list
  end

  def unsubscribed
    remove_user_presence
    broadcast_presence_list
  end

  def heartbeat
    # Client sends this periodically to confirm still connected
    refresh_user_presence
  end

  private

  def redis
    @redis ||= Redis.new
  end

  def presence_key
    "presence:online_users"
  end

  def add_user_presence
    redis.hset(presence_key, current_user.id, Time.current.to_i)
  end

  def remove_user_presence
    redis.hdel(presence_key, current_user.id)
  end

  def refresh_user_presence
    redis.hset(presence_key, current_user.id, Time.current.to_i)
    cleanup_stale_users
  end

  def cleanup_stale_users
    # Remove users not seen in 30 seconds
    threshold = 30.seconds.ago.to_i
    redis.hgetall(presence_key).each do |user_id, timestamp|
      if timestamp.to_i < threshold
        redis.hdel(presence_key, user_id)
      end
    end
  end

  def online_user_ids
    redis.hkeys(presence_key).map(&:to_i)
  end

  def broadcast_presence_list
    users = User.where(id: online_user_ids)
                .select(:id, :name, :avatar_url)

    ActionCable.server.broadcast("presence", {
      users: users.as_json,
      count: users.size
    })
  end
end
```

## Client-Side Presence

```javascript
// app/javascript/channels/presence_channel.js
import consumer from "./consumer"

consumer.subscriptions.create("PresenceChannel", {
  connected() {
    // Start heartbeat
    this.heartbeatInterval = setInterval(() => {
      this.perform("heartbeat")
    }, 15000) // Every 15 seconds
  },

  disconnected() {
    clearInterval(this.heartbeatInterval)
  },

  received(data) {
    this.updateOnlineUsers(data.users, data.count)
  },

  updateOnlineUsers(users, count) {
    const container = document.getElementById("online-users")
    if (!container) return

    container.innerHTML = `
      <h4>Online (${count})</h4>
      <ul>
        ${users.map(user => `
          <li class="online-user" data-user-id="${user.id}">
            <span class="status-dot online"></span>
            ${user.name}
          </li>
        `).join("")}
      </ul>
    `
  }
})
```

## Room-Specific Presence

```ruby
class RoomPresenceChannel < ApplicationCable::Channel
  def subscribed
    @room = Room.find(params[:room_id])
    stream_from "room_presence_#{@room.id}"
    add_to_room
  end

  def unsubscribed
    remove_from_room if @room
  end

  private

  def add_to_room
    redis.sadd("room:#{@room.id}:users", current_user.id)
    broadcast_room_presence
  end

  def remove_from_room
    redis.srem("room:#{@room.id}:users", current_user.id)
    broadcast_room_presence
  end
end
```

## Resources

- [Action Cable Connections](https://guides.rubyonrails.org/action_cable_overview.html#connections) — Managing connections and presence

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*