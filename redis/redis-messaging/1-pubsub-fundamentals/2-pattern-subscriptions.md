---
source_course: "redis-messaging"
source_lesson: "redis-messaging-pattern-subscriptions"
---

# Pattern Subscriptions

Instead of subscribing to specific channels, you can subscribe to patterns that match multiple channels using glob-style wildcards.

## PSUBSCRIBE

```redis
# Subscribe to all channels starting with "news:"
PSUBSCRIBE news:*

# Matches: news:sports, news:politics, news:tech
# Doesn't match: news, sports:news
```

## Pattern Wildcards

| Pattern | Matches |
|---------|--------|
| `*` | Any sequence of characters |
| `?` | Exactly one character |
| `[abc]` | One of the characters a, b, or c |

### Examples

```redis
# Match all channels
PSUBSCRIBE *

# Match news from any country
PSUBSCRIBE news:*:breaking
# Matches: news:us:breaking, news:uk:breaking

# Match single character variations
PSUBSCRIBE user:?:online
# Matches: user:1:online, user:a:online
# Doesn't match: user:123:online

# Match specific options
PSUBSCRIBE alert:[123]
# Matches: alert:1, alert:2, alert:3
```

## Pattern Message Format

Messages from pattern subscriptions have a different format:

```redis
# Regular subscription message:
1) "message"
2) "news"              # channel name
3) "Hello World"       # message

# Pattern subscription message:
1) "pmessage"
2) "news:*"            # pattern that matched
3) "news:sports"       # actual channel
4) "Game update!"      # message
```

## Combining Regular and Pattern Subscriptions

```redis
# Subscribe to both specific channels and patterns
SUBSCRIBE alerts
PSUBSCRIBE news:*
```

**Note**: If a channel matches both a regular subscription and a pattern, you'll receive the message twice!

```redis
# If subscribed to both:
SUBSCRIBE news:sports
PSUBSCRIBE news:*

# Publishing to news:sports sends TWO messages to this client
```

## PUNSUBSCRIBE

```redis
# Unsubscribe from specific patterns
PUNSUBSCRIBE news:*

# Unsubscribe from all patterns
PUNSUBSCRIBE
```

## Performance Considerations

Pattern matching has overhead:

```
┌─────────────────────────────────────────────────────────────┐
│  Regular subscription:                                      │
│  - O(1) lookup per channel                                 │
│  - Very fast                                               │
├─────────────────────────────────────────────────────────────┤
│  Pattern subscription:                                      │
│  - O(N) where N = number of pattern subscriptions          │
│  - Each message must check against all patterns            │
│  - Use sparingly with many patterns                        │
└─────────────────────────────────────────────────────────────┘
```

## Practical Example: Multi-Tenant Notifications

```redis
# Subscribe to all notifications for tenant "acme"
PSUBSCRIBE acme:notifications:*

# Different notification types publish to:
PUBLISH acme:notifications:orders "New order #123"
PUBLISH acme:notifications:users "New user signup"
PUBLISH acme:notifications:alerts "Server warning"

# Subscriber receives all three
```

📖 [PSUBSCRIBE Command](https://redis.io/docs/latest/commands/psubscribe/)

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*