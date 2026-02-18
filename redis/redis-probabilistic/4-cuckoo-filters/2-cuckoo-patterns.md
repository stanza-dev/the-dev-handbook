---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-cuckoo-patterns"
---

# Cuckoo Filter Use Cases

Practical patterns for using Cuckoo filters in production.

## Session Management

```python
import redis
import uuid
from datetime import datetime

r = redis.Redis()

class SessionManager:
    def __init__(self, capacity=1000000):
        self.filter_key = 'active:sessions'
        try:
            r.execute_command('CF.RESERVE', self.filter_key, capacity)
        except redis.ResponseError:
            pass  # Already exists
    
    def create_session(self, user_id):
        """Create a new session"""
        session_id = str(uuid.uuid4())
        
        # Store full session data
        r.hset(f'session:{session_id}', mapping={
            'user_id': user_id,
            'created': str(datetime.now())
        })
        r.expire(f'session:{session_id}', 3600)
        
        # Add to Cuckoo filter for quick checks
        r.execute_command('CF.ADD', self.filter_key, session_id)
        
        return session_id
    
    def is_valid_session(self, session_id):
        """Quick check if session might be valid"""
        # Fast Cuckoo check first
        if not r.execute_command('CF.EXISTS', self.filter_key, session_id):
            return False  # Definitely invalid
        
        # Slower but definitive check
        return r.exists(f'session:{session_id}')
    
    def destroy_session(self, session_id):
        """Remove session"""
        r.delete(f'session:{session_id}')
        r.execute_command('CF.DEL', self.filter_key, session_id)
```

## Reservation System

```python
class ReservationSystem:
    def __init__(self):
        self.filter_key = 'reservations:filter'
        try:
            r.execute_command('CF.RESERVE', self.filter_key, 100000)
        except:
            pass
    
    def reserve(self, resource_id, user_id):
        """Reserve a resource"""
        key = f'{resource_id}:{user_id}'
        
        # Check if already reserved (quick check)
        if r.execute_command('CF.EXISTS', self.filter_key, resource_id):
            return {'error': 'Resource might be reserved'}
        
        # Make reservation
        r.execute_command('CF.ADD', self.filter_key, resource_id)
        r.setex(f'reservation:{key}', 900, user_id)  # 15 min hold
        
        return {'success': True, 'expires_in': 900}
    
    def release(self, resource_id):
        """Release a reservation"""
        # Delete from filter (allows re-reservation)
        r.execute_command('CF.DEL', self.filter_key, resource_id)
        # Delete actual reservation keys...
```

## CF.COUNT: Approximate Counting

```redis
# Cuckoo can count item occurrences (approximately)
CF.ADD items "apple"
CF.ADD items "apple"
CF.ADD items "apple"

CF.COUNT items "apple"
# Returns: 3 (approximate)
```

## Caution: Over-Deletion

Deleting an item that was never added might cause future false negatives:

```python
def safe_delete(filter_key, item):
    """Only delete if item exists in filter"""
    if r.execute_command('CF.EXISTS', filter_key, item):
        r.execute_command('CF.DEL', filter_key, item)
        return True
    return False
```

## Choosing Between Bloom and Cuckoo

| Scenario | Recommendation |
|----------|----------------|
| Write-once data (URLs, IDs) | Bloom Filter |
| Temporary data (sessions) | Cuckoo Filter |
| Need deletion | Cuckoo Filter |
| Minimum memory | Bloom Filter |
| Need counting | Cuckoo Filter |

📖 [Cuckoo Filter](https://redis.io/docs/latest/develop/data-types/probabilistic/cuckoo-filter/)

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*