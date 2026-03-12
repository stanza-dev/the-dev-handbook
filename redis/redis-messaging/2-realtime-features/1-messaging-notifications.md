---
source_course: "redis-messaging"
source_lesson: "redis-messaging-notifications"
---

# Real-Time Notifications

## Introduction

Push notifications are one of the most natural use cases for Redis Pub/Sub. A backend event publishes to a user-specific channel, and any connected client subscribed to that channel receives the notification immediately.

## Key Concepts

- **User-specific channel**: name channels with the user ID (e.g., `notifications:user:42`) so each user only receives their own notifications
- **Group channel**: a shared channel (e.g., `notifications:team:engineering`) for broadcasting to all members of a team
- **Dual write**: persisting a notification to a database alongside the Pub/Sub publish, so offline users see it on next login

## Real World Context

- GitHub: push event triggers a publish to all users watching the repository
- Slack: incoming message publishes to the channel's subscribers
- E-commerce: order status change publishes to the customer's notification channel

## Deep Dive

```bash
# User 42's session subscribes on login
SUBSCRIBE notifications:user:42

# Backend publishes when something happens
PUBLISH notifications:user:42 '{"type":"order_shipped","orderId":"ord-99"}'
# => (integer) 1  <- user is online and received it

# Broadcast to a team channel
PUBLISH notifications:team:engineering "Deployment complete"
# All subscribed team members receive it
```

Because Pub/Sub does not persist messages, notifications sent while a user is offline are lost. Common strategies:

1. **Dual write**: also save the notification to a database and show it on next login
2. **Streams**: publish to a per-user Stream so offline users can catch up on reconnect
3. **Polling fallback**: periodically query a notifications table

## Common Pitfalls

- **Not handling offline users**: Pub/Sub has no queue; users who disconnect miss all notifications sent during downtime
- **Putting full payloads in messages**: large JSON payloads increase memory pressure; prefer small trigger messages with an ID to fetch from cache
- **Using one shared channel for all users**: users would receive each other's private notifications

## Best Practices

- Always use a per-user channel namespace (`notifications:user:{id}`) for private notifications
- Pair Pub/Sub with a persistent notifications table for at-least-once delivery to offline users
- Use group channels for team/room broadcasts but keep them namespaced (`notifications:team:{id}`)

## Summary

User-scoped channel naming (e.g., `notifications:user:{id}`) enables targeted real-time push. Always pair Pub/Sub notifications with a persistent store for offline users who cannot receive live messages.

## Code Examples

**User notifications**

```bash
# Subscribe (runs in user's WebSocket handler)
SUBSCRIBE notifications:user:42

# Backend event triggers publish
PUBLISH notifications:user:42 '{"type":"like","postId":"p-7"}'
# => (integer) 1

# Broadcast to a group
PUBLISH notifications:team:ops "Alert: high memory usage"
# => (integer) 3  <- 3 ops members are online
```


## Resources

- [Redis Pub/Sub](https://redis.io/docs/latest/develop/interact/pubsub/) — Official Redis Pub/Sub documentation

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*