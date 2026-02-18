---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-websockets-intro"
---

# Understanding WebSockets

WebSockets enable real-time, bidirectional communication between clients and servers, unlike traditional HTTP request-response cycles.

## HTTP vs WebSockets

### Traditional HTTP

```
Client                  Server
  │                       │
  │──── Request ─────────>│
  │<─── Response ─────────│
  │                       │
  │──── Request ─────────>│
  │<─── Response ─────────│
```

- Request-response model
- Connection closes after each response
- Client initiates all communication
- Server cannot push data to client

### WebSockets

```
Client                  Server
  │                       │
  │── Upgrade Request ───>│
  │<── Upgrade Response ──│
  │                       │
  │<══ Persistent ═══════>│
  │<══ Connection ═══════>│
  │                       │
  │──── Message ─────────>│
  │<─── Message ──────────│
  │<─── Message ──────────│
  │──── Message ─────────>│
```

- Persistent, full-duplex connection
- Either side can send messages anytime
- Lower latency than polling
- Perfect for real-time features

## When to Use WebSockets

**Good use cases:**
- Chat applications
- Live notifications
- Real-time dashboards
- Collaborative editing
- Live sports scores
- Multiplayer games
- Presence indicators ("User is typing...")

**Not ideal for:**
- Simple CRUD operations
- Infrequent updates
- One-time data fetching

## Action Cable Overview

Action Cable integrates WebSockets into Rails:

```
┌─────────────────────────────────────────────────────────┐
│                    Action Cable                          │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────┐    ┌──────────────┐                   │
│  │  Connection  │────│   Channel    │                   │
│  │  (1 per tab) │    │  (Topic)     │                   │
│  └──────────────┘    └──────────────┘                   │
│         │                   │                            │
│         │            ┌──────────────┐                   │
│         └────────────│ Subscription │                   │
│                      │  (Listener)  │                   │
│                      └──────────────┘                   │
│                             │                            │
│                      ┌──────────────┐                   │
│                      │ Broadcasting │                   │
│                      │  (Messages)  │                   │
│                      └──────────────┘                   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

**Key concepts:**
- **Connection**: One WebSocket connection per browser tab
- **Consumer**: The client-side WebSocket client
- **Channel**: Like a controller for WebSocket messages
- **Subscription**: Client subscribing to a channel
- **Broadcasting**: Sending messages to subscribers

## The Pub/Sub Pattern

Action Cable uses publish/subscribe:

1. **Publisher** broadcasts messages to a named stream
2. **Subscribers** receive messages from streams they're subscribed to
3. Redis (or Postgres) handles message distribution

```ruby
# Publishing
ActionCable.server.broadcast("chat_room_1", { message: "Hello!" })

# Subscribing (in channel)
stream_from "chat_room_1"
```

## Resources

- [Action Cable Overview](https://guides.rubyonrails.org/action_cable_overview.html) — Official guide to Action Cable

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*