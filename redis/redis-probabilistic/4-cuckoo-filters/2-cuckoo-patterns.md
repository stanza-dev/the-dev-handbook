---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-cuckoo-patterns"
---

# Cuckoo Filter Use Cases

## Introduction

Cuckoo filters earn their keep in systems where items come and go — sessions, reservations, feature rollouts, and anywhere you need to remove entries without rebuilding the entire filter. This lesson walks through production patterns that exploit Cuckoo filters' unique ability to delete.

## Key Concepts

- **Ephemeral Membership**: Items that are added and later removed (sessions, locks, reservations)
- **Pre-filter Gate**: Using CF.EXISTS as a fast first check before hitting slower storage
- **Safe Deletion**: Always verify existence before deleting to avoid removing a fingerprint that belongs to a different item
- **CF.COUNT**: Returns an approximate count of how many times an item has been added (minus deletions)

## Real World Context

A ticket reservation system must track which seats are held in real time. Seats are reserved (added) and released (deleted) hundreds of times per second. A Cuckoo filter gives sub-millisecond checks for "is this seat taken?" while supporting immediate removal when a reservation expires — something a Bloom filter simply cannot do.

## Deep Dive

### Pattern 1: Session Management

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
            pass

    def create_session(self, user_id):
        session_id = str(uuid.uuid4())
        r.hset(f'session:{session_id}', mapping={
            'user_id': user_id,
            'created': str(datetime.now())
        })
        r.expire(f'session:{session_id}', 3600)
        r.execute_command('CF.ADD', self.filter_key, session_id)
        return session_id

    def is_valid_session(self, session_id):
        if not r.execute_command('CF.EXISTS', self.filter_key, session_id):
            return False
        return r.exists(f'session:{session_id}')

    def destroy_session(self, session_id):
        r.delete(f'session:{session_id}')
        r.execute_command('CF.DEL', self.filter_key, session_id)
```

### Pattern 2: Reservation System

```python
class ReservationSystem:
    def __init__(self):
        self.filter_key = 'reservations:filter'
        try:
            r.execute_command('CF.RESERVE', self.filter_key, 100000)
        except redis.ResponseError:
            pass

    def reserve(self, resource_id, user_id):
        if r.execute_command('CF.EXISTS', self.filter_key, resource_id):
            return {'error': 'Resource might be reserved'}
        r.execute_command('CF.ADD', self.filter_key, resource_id)
        key = f'{resource_id}:{user_id}'
        r.setex(f'reservation:{key}', 900, user_id)
        return {'success': True, 'expires_in': 900}

    def release(self, resource_id):
        r.execute_command('CF.DEL', self.filter_key, resource_id)
```

### Pattern 3: Feature Flag Rollout

```python
class FeatureGate:
    def __init__(self, feature_name, capacity=100000):
        self.filter_key = f'feature:{feature_name}:users'
        try:
            r.execute_command('CF.RESERVE', self.filter_key, capacity)
        except redis.ResponseError:
            pass

    def enable_for_user(self, user_id):
        r.execute_command('CF.ADDNX', self.filter_key, user_id)

    def disable_for_user(self, user_id):
        if r.execute_command('CF.EXISTS', self.filter_key, user_id):
            r.execute_command('CF.DEL', self.filter_key, user_id)

    def is_enabled(self, user_id):
        return r.execute_command('CF.EXISTS', self.filter_key, user_id) == 1
```

### Safe Deletion Pattern

Deleting an item that was never added can corrupt the filter. Always guard deletions:

```python
def safe_delete(filter_key, item):
    """Only delete if item exists in filter."""
    if r.execute_command('CF.EXISTS', filter_key, item):
        r.execute_command('CF.DEL', filter_key, item)
        return True
    return False
```

### Choosing Between Bloom and Cuckoo

| Scenario | Recommendation |
|----------|----------------|
| Write-once data (URLs, IDs) | Bloom Filter |
| Temporary data (sessions) | Cuckoo Filter |
| Need deletion | Cuckoo Filter |
| Minimum memory | Bloom Filter |
| Need counting | Cuckoo Filter |

## Common Pitfalls

1. **Deleting without checking existence** — Calling CF.DEL on an item that was never added can remove a fingerprint belonging to a different item, introducing false negatives. Always check CF.EXISTS first.
2. **Using CF.ADD when CF.ADDNX is more appropriate** — CF.ADD allows duplicate fingerprints for the same item, inflating CF.COUNT. Use CF.ADDNX if each item should only be counted once.

## Best Practices

1. **Track filter utilization with CF.INFO** — Monitor `Number of items inserted` vs `Capacity` and `Number of filters`. Alert when utilization exceeds 80% so you can plan a capacity increase.
2. **Use Cuckoo filters for ephemeral data, Bloom for permanent** — If items are never deleted (e.g., crawled URLs), Bloom filters are smaller and simpler. Reserve Cuckoo filters for data with a lifecycle.

## Summary

- Session management: CF.ADD on login, CF.DEL on logout, CF.EXISTS for fast validation
- Reservations: CF.ADD to hold, CF.DEL to release, CF.EXISTS to check availability
- Feature flags: CF.ADDNX to enable, safe CF.DEL to disable, CF.EXISTS to gate access
- Always guard CF.DEL with a CF.EXISTS check to prevent corrupting the filter

## Code Examples

**Feature flag rollout with Cuckoo filter — enable, disable, and check per user**

```python
import redis

r = redis.Redis()

class FeatureGate:
    """Feature flag rollout using Cuckoo filter."""
    def __init__(self, feature_name, capacity=100000):
        self.filter_key = f'feature:{feature_name}:users'
        try:
            r.execute_command('CF.RESERVE', self.filter_key, capacity)
        except redis.ResponseError:
            pass

    def enable_for_user(self, user_id):
        r.execute_command('CF.ADDNX', self.filter_key, user_id)

    def disable_for_user(self, user_id):
        if r.execute_command('CF.EXISTS', self.filter_key, user_id):
            r.execute_command('CF.DEL', self.filter_key, user_id)

    def is_enabled(self, user_id):
        return r.execute_command('CF.EXISTS', self.filter_key, user_id) == 1

# Usage
gate = FeatureGate('dark_mode')
gate.enable_for_user('user:1001')
print(gate.is_enabled('user:1001'))  # True
gate.disable_for_user('user:1001')
print(gate.is_enabled('user:1001'))  # False
```


## Resources

- [Cuckoo Filters](https://redis.io/docs/latest/develop/data-types/probabilistic/cuckoo-filter/) — Redis Cuckoo Filter documentation
- [CF.DEL Command](https://redis.io/docs/latest/commands/cf.del/) — CF.DEL command reference for item deletion

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*