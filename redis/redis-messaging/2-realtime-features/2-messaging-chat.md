---
source_course: "redis-messaging"
source_lesson: "redis-messaging-chat"
---

# Building a Chat System

Let's build a multi-room chat system with Redis Pub/Sub.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  User A ──────────────┐                                     │
│  User B ──────────────┤                                     │
│  User C ──────────────┼──▶ Room: general ──▶ All Users     │
│                       │                                     │
│  User A ──────────────┼──▶ Room: dev-team ──▶ A, B only    │
│  User B ──────────────┘                                     │
└─────────────────────────────────────────────────────────────┘
```

## Channel Structure

```redis
# Chat room channels
chat:room:general
chat:room:dev-team
chat:room:random

# Direct messages (sorted user IDs)
chat:dm:1001:1002
chat:dm:1001:1003
```

## Message Format

```python
def create_message(user_id, username, content, room=None):
    return {
        'id': str(uuid.uuid4()),
        'user_id': user_id,
        'username': username,
        'content': content,
        'room': room,
        'timestamp': datetime.utcnow().isoformat(),
        'type': 'message'
    }
```

## Joining and Leaving Rooms

```python
class ChatRoom:
    def __init__(self, redis_client, room_name):
        self.r = redis_client
        self.room = room_name
        self.channel = f"chat:room:{room_name}"
        self.members_key = f"chat:room:{room_name}:members"
    
    def join(self, user_id, username):
        # Add to room members
        self.r.sadd(self.members_key, user_id)
        
        # Announce join
        self.r.publish(self.channel, json.dumps({
            'type': 'system',
            'content': f"{username} joined the room",
            'timestamp': datetime.utcnow().isoformat()
        }))
    
    def leave(self, user_id, username):
        self.r.srem(self.members_key, user_id)
        
        self.r.publish(self.channel, json.dumps({
            'type': 'system',
            'content': f"{username} left the room",
            'timestamp': datetime.utcnow().isoformat()
        }))
    
    def send_message(self, user_id, username, content):
        message = create_message(user_id, username, content, self.room)
        
        # Publish to real-time listeners
        self.r.publish(self.channel, json.dumps(message))
        
        # Store for history (last 1000 messages)
        history_key = f"chat:room:{self.room}:history"
        self.r.lpush(history_key, json.dumps(message))
        self.r.ltrim(history_key, 0, 999)
    
    def get_history(self, count=50):
        messages = self.r.lrange(
            f"chat:room:{self.room}:history", 0, count-1
        )
        return [json.loads(m) for m in messages]
    
    def get_members(self):
        return self.r.smembers(self.members_key)
```

## Direct Messages

```python
class DirectMessage:
    def __init__(self, redis_client, user1_id, user2_id):
        self.r = redis_client
        # Always sort IDs for consistent channel name
        ids = sorted([user1_id, user2_id])
        self.channel = f"chat:dm:{ids[0]}:{ids[1]}"
        self.history_key = f"chat:dm:{ids[0]}:{ids[1]}:history"
    
    def send(self, from_user_id, from_username, content):
        message = create_message(from_user_id, from_username, content)
        
        self.r.publish(self.channel, json.dumps(message))
        self.r.lpush(self.history_key, json.dumps(message))
        self.r.ltrim(self.history_key, 0, 499)
    
    def get_history(self, count=50):
        messages = self.r.lrange(self.history_key, 0, count-1)
        return [json.loads(m) for m in messages]
```

## Typing Indicators

```python
def send_typing_indicator(user_id, room):
    r.publish(f"chat:room:{room}", json.dumps({
        'type': 'typing',
        'user_id': user_id,
        'timestamp': datetime.utcnow().isoformat()
    }))

# Client handles typing indicator with timeout
# Clear after 3 seconds of no typing updates
```

## Message Read Receipts

```python
def mark_messages_read(user_id, room, last_read_id):
    r.hset(f"chat:room:{room}:read_receipts", user_id, last_read_id)
    r.publish(f"chat:room:{room}", json.dumps({
        'type': 'read_receipt',
        'user_id': user_id,
        'last_read': last_read_id
    }))
```

📖 [Chat Application Pattern](https://redis.io/docs/latest/develop/use/patterns/)

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*