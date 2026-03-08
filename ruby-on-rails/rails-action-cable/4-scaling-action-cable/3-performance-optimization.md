---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-performance-optimization"
---

# Performance Optimization

## Introduction
As your application scales, inefficient Action Cable usage can become a bottleneck. Optimizing broadcast frequency, payload size, and subscription management keeps your real-time features responsive without overwhelming the server or client.

## Key Concepts
- **Broadcast Debouncing**: Reducing the frequency of broadcasts by batching or throttling updates, especially for rapidly changing data.
- **Payload Compression**: Minimizing the size of broadcast messages to reduce bandwidth usage and improve delivery speed.
- **Subscription Scoping**: Ensuring clients subscribe only to the streams they actively need, reducing the number of messages processed per connection.
- **Background Job Broadcasting**: Moving broadcast logic to background jobs to avoid blocking the request cycle.

## Real World Context
Imagine a live dashboard showing real-time order counts. If every new order triggers a broadcast, a flash sale generating 100 orders per second produces 100 broadcasts per second to every dashboard viewer. By debouncing — broadcasting the updated count every 2 seconds instead of on every order — you reduce traffic by 99% with minimal impact on perceived real-time accuracy.

## Deep Dive
### Debouncing Broadcasts

For data that changes frequently, batch updates in a background job:

```ruby
# app/jobs/dashboard_broadcast_job.rb
class DashboardBroadcastJob < ApplicationJob
  def perform(dashboard_id)
    dashboard = Dashboard.find(dashboard_id)

    DashboardChannel.broadcast_to(dashboard, {
      type: "stats_update",
      data: {
        order_count: dashboard.orders.today.count,
        revenue: dashboard.revenue_today,
        active_users: dashboard.active_user_count
      }
    })
  end
end

# Schedule periodically using Solid Queue or a recurring job
# Instead of broadcasting on every order creation
```

### Minimizing Payload Size

Every byte sent through a WebSocket is multiplied by the number of subscribers. Keep payloads lean:

```ruby
# Bad: 2KB payload with unnecessary data
ChatChannel.broadcast_to(room, message.as_json(include: { user: { include: :profile } }))

# Good: 200-byte payload with only what the UI needs
ChatChannel.broadcast_to(room, {
  type: "msg",
  id: message.id,
  body: message.body,
  uid: message.user_id,
  name: message.user.username,
  at: message.created_at.to_i
})
```

For even smaller payloads, use short key names and Unix timestamps instead of ISO 8601 strings.

### Managing Subscriptions

Clean up subscriptions when they are no longer needed:

```ruby
class ChatChannel < ApplicationCable::Channel
  def subscribed
    @room = Room.find(params[:room_id])
    stream_for @room

    # Track presence
    current_user.update!(last_seen_in_room: @room.id)
  end

  def unsubscribed
    # Clean up presence tracking
    current_user.update!(last_seen_in_room: nil) if @room
    stop_all_streams
  end
end
```

The `stop_all_streams` method in `unsubscribed` explicitly cleans up all streams for the subscription. While Action Cable calls this automatically, being explicit makes your intent clear.

### N+1 Query Prevention

When broadcasting involves database queries, watch for N+1 patterns:

```ruby
# Bad: N+1 queries in a broadcast loop
room.users.each do |user|
  NotificationsChannel.broadcast_to(user, { message: "New update" })
end

# Better: Single broadcast to the room stream
RoomChannel.broadcast_to(room, { type: "update", message: "New update" })
```

## Common Pitfalls
1. **Broadcasting on every database change** — High-frequency updates like counters or timestamps should be batched or debounced, not broadcast individually.
2. **Leaving stale subscriptions open** — If a user navigates away from a page but the subscription is not cleaned up, they continue receiving (and ignoring) broadcasts, wasting resources.
3. **Querying the database inside `received` on the server** — Channel actions should be lean. Move complex queries to background jobs.

## Best Practices
1. **Debounce high-frequency broadcasts** — For data that changes multiple times per second, broadcast on a timer (every 1-5 seconds) rather than on every change.
2. **Use the smallest possible payload** — Every byte is multiplied by every subscriber. Short keys, Unix timestamps, and minimal nesting reduce bandwidth.
3. **Unsubscribe clients when they navigate away** — Use Turbo's lifecycle hooks or `disconnected` callbacks to clean up subscriptions that are no longer visible.

## Summary
- Debounce broadcasts for rapidly changing data to reduce message volume by orders of magnitude.
- Minimize payload size — every byte is multiplied by the subscriber count.
- Clean up subscriptions in `unsubscribed` to free server resources.
- Move broadcast logic to background jobs to keep the request cycle fast.
- Watch for N+1 queries in broadcast loops — prefer single stream broadcasts over per-user loops.

## Code Examples

**Debounced broadcasting — a periodic job broadcasts aggregated stats instead of triggering on every individual change**

```ruby
# Debounced broadcasting via a periodic background job
class DashboardBroadcastJob < ApplicationJob
  def perform(dashboard_id)
    dashboard = Dashboard.find(dashboard_id)
    DashboardChannel.broadcast_to(dashboard, {
      type: "stats_update",
      data: {
        order_count: dashboard.orders.today.count,
        revenue: dashboard.revenue_today
      }
    })
  end
end

# Run every 2 seconds instead of on every order
```


## Resources

- [Action Cable Performance](https://guides.rubyonrails.org/action_cable_overview.html#configuration) — Configuration options that affect Action Cable performance

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*