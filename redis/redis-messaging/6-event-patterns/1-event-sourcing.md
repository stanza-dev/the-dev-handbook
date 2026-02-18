---
source_course: "redis-messaging"
source_lesson: "redis-messaging-event-sourcing"
---

# Event Sourcing with Streams

Event sourcing stores all changes as a sequence of events. Redis Streams is an excellent fit for this pattern.

## Event Sourcing Concept

```
┌─────────────────────────────────────────────────────────────┐
│  Traditional: Store current state                           │
│  User { id: 1, name: "Alice", email: "new@example.com" }   │
├─────────────────────────────────────────────────────────────┤
│  Event Sourcing: Store all events                           │
│  UserCreated { id: 1, name: "Alice", email: "a@x.com" }    │
│  EmailChanged { id: 1, email: "alice@example.com" }        │
│  EmailChanged { id: 1, email: "new@example.com" }          │
│                                                             │
│  Current state = replay all events                         │
└─────────────────────────────────────────────────────────────┘
```

## Benefits

- **Complete audit trail**: Every change is recorded
- **Time travel**: Reconstruct state at any point in time
- **Debugging**: See exactly what happened and when
- **Analytics**: Analyze patterns in historical data

## Implementing Event Sourcing

### Event Structure

```python
import json
import uuid
from datetime import datetime

def create_event(aggregate_type, aggregate_id, event_type, data):
    return {
        'event_id': str(uuid.uuid4()),
        'aggregate_type': aggregate_type,
        'aggregate_id': str(aggregate_id),
        'event_type': event_type,
        'data': json.dumps(data),
        'timestamp': datetime.utcnow().isoformat(),
        'version': '1'
    }
```

### Event Store

```python
class EventStore:
    def __init__(self, redis_client):
        self.r = redis_client
    
    def append(self, aggregate_type, aggregate_id, event_type, data):
        """Append an event to the event stream"""
        # Global event stream
        stream_key = f"events:{aggregate_type}"
        
        event = create_event(aggregate_type, aggregate_id, event_type, data)
        
        entry_id = self.r.xadd(stream_key, event, maxlen=100000)
        
        # Also index by aggregate ID for fast lookups
        self.r.xadd(f"events:{aggregate_type}:{aggregate_id}", event)
        
        return entry_id
    
    def get_events(self, aggregate_type, aggregate_id, after_id=None):
        """Get all events for an aggregate"""
        stream_key = f"events:{aggregate_type}:{aggregate_id}"
        start = after_id or '-'
        
        events = self.r.xrange(stream_key, start, '+')
        return [
            {'id': e[0], **e[1], 'data': json.loads(e[1]['data'])}
            for e in events
        ]
    
    def replay_to_state(self, aggregate_type, aggregate_id, handlers):
        """Rebuild current state by replaying events"""
        events = self.get_events(aggregate_type, aggregate_id)
        state = {}
        
        for event in events:
            event_type = event['event_type']
            if event_type in handlers:
                state = handlers[event_type](state, event['data'])
        
        return state
```

### Example: User Aggregate

```python
event_store = EventStore(redis_client)

# Event handlers for rebuilding state
user_handlers = {
    'UserCreated': lambda state, data: {
        **state,
        'id': data['id'],
        'name': data['name'],
        'email': data['email'],
        'created_at': data['timestamp']
    },
    'EmailChanged': lambda state, data: {
        **state,
        'email': data['email'],
        'email_updated_at': data['timestamp']
    },
    'NameChanged': lambda state, data: {
        **state,
        'name': data['name']
    }
}

# Create a user
event_store.append('user', 1, 'UserCreated', {
    'id': 1,
    'name': 'Alice',
    'email': 'alice@example.com',
    'timestamp': datetime.utcnow().isoformat()
})

# Change email
event_store.append('user', 1, 'EmailChanged', {
    'email': 'alice.smith@example.com',
    'timestamp': datetime.utcnow().isoformat()
})

# Get current state
user_state = event_store.replay_to_state('user', 1, user_handlers)
print(user_state)
# {'id': 1, 'name': 'Alice', 'email': 'alice.smith@example.com', ...}

# Get complete history
history = event_store.get_events('user', 1)
```

## Snapshots for Performance

Replaying thousands of events is slow. Use periodic snapshots:

```python
def save_snapshot(aggregate_type, aggregate_id, state, version_id):
    key = f"snapshot:{aggregate_type}:{aggregate_id}"
    r.hset(key, mapping={
        'state': json.dumps(state),
        'version_id': version_id
    })

def load_with_snapshot(aggregate_type, aggregate_id, handlers):
    snapshot_key = f"snapshot:{aggregate_type}:{aggregate_id}"
    snapshot = r.hgetall(snapshot_key)
    
    if snapshot:
        state = json.loads(snapshot['state'])
        after_id = snapshot['version_id']
    else:
        state = {}
        after_id = None
    
    # Apply events after snapshot
    events = event_store.get_events(aggregate_type, aggregate_id, after_id)
    for event in events:
        handler = handlers.get(event['event_type'])
        if handler:
            state = handler(state, event['data'])
    
    return state
```

📖 [Event Sourcing Pattern](https://redis.io/docs/latest/develop/use/patterns/)

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*