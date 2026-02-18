---
source_course: "redis-messaging"
source_lesson: "redis-messaging-notifications"
---

# Real-Time Notifications

Let's build practical notification systems with Redis Pub/Sub.

## User Notification System

### Channel Naming Convention

```redis
# Per-user notification channels
user:1001:notifications
user:1001:notifications:urgent
user:1001:notifications:mentions

# Broadcast channels
notifications:system
notifications:maintenance
```

### Publisher: Sending Notifications

```python
import redis
import json
from datetime import datetime

r = redis.Redis(decode_responses=True)

def send_notification(user_id, notification_type, data):
    notification = {
        'type': notification_type,
        'data': data,
        'timestamp': datetime.utcnow().isoformat(),
        'id': str(uuid.uuid4())
    }
    
    channel = f"user:{user_id}:notifications"
    r.publish(channel, json.dumps(notification))
    
    # Also store for users who are offline
    r.lpush(f"user:{user_id}:notifications:unread", json.dumps(notification))
    r.ltrim(f"user:{user_id}:notifications:unread", 0, 99)  # Keep last 100

# Send notification
send_notification(1001, 'new_message', {
    'from': 'Alice',
    'preview': 'Hey, are you free tomorrow?'
})
```

### Subscriber: Receiving Notifications

```python
import redis
import json

r = redis.Redis(decode_responses=True)
pubsub = r.pubsub()

def notification_handler(message):
    if message['type'] == 'message':
        notification = json.loads(message['data'])
        print(f"New notification: {notification['type']}")
        # Update UI, play sound, etc.

pubsub.subscribe(**{'user:1001:notifications': notification_handler})

# Run in background thread
thread = pubsub.run_in_thread(sleep_time=0.001)
```

## Handling Offline Users

Pub/Sub messages are lost if the user is offline. Combine with persistence:

```python
def get_unread_notifications(user_id):
    """Get notifications missed while offline"""
    key = f"user:{user_id}:notifications:unread"
    notifications = r.lrange(key, 0, -1)
    return [json.loads(n) for n in notifications]

def mark_notifications_read(user_id):
    """Clear unread notifications after user views them"""
    r.delete(f"user:{user_id}:notifications:unread")
```

## Broadcast Notifications

System-wide announcements:

```python
def broadcast_maintenance(message, scheduled_time):
    notification = {
        'type': 'maintenance',
        'message': message,
        'scheduled': scheduled_time,
        'priority': 'high'
    }
    r.publish('notifications:system', json.dumps(notification))

# All connected users subscribe to system channel
pubsub.subscribe('notifications:system')
pubsub.subscribe(f'user:{current_user_id}:notifications')
```

## Presence System

Track online users:

```python
def user_came_online(user_id):
    # Add to online users set
    r.sadd('users:online', user_id)
    # Publish presence update
    r.publish('presence:updates', json.dumps({
        'user_id': user_id,
        'status': 'online'
    }))

def user_went_offline(user_id):
    r.srem('users:online', user_id)
    r.publish('presence:updates', json.dumps({
        'user_id': user_id,
        'status': 'offline'
    }))

def get_online_friends(user_id):
    friends = r.smembers(f'user:{user_id}:friends')
    online_users = r.smembers('users:online')
    return friends.intersection(online_users)
```

## WebSocket Integration

```python
# FastAPI + WebSocket example
from fastapi import FastAPI, WebSocket
import asyncio

app = FastAPI()

@app.websocket("/ws/{user_id}")
async def websocket_endpoint(websocket: WebSocket, user_id: int):
    await websocket.accept()
    
    pubsub = r.pubsub()
    pubsub.subscribe(f'user:{user_id}:notifications')
    pubsub.subscribe('notifications:system')
    
    try:
        while True:
            message = pubsub.get_message(ignore_subscribe_messages=True)
            if message:
                await websocket.send_text(message['data'])
            await asyncio.sleep(0.01)
    finally:
        pubsub.unsubscribe()
```

📖 [Building Real-Time Apps](https://redis.io/docs/latest/develop/pubsub/)

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*