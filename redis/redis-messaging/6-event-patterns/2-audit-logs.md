---
source_course: "redis-messaging"
source_lesson: "redis-messaging-audit-logs"
---

# Audit Logging with Streams

Streams are ideal for audit logs: they're append-only, ordered, and support efficient range queries.

## Audit Log Structure

```redis
# Add audit entry
XADD audit:actions * \
    timestamp "2024-01-15T10:30:00Z" \
    user_id "1001" \
    action "user.login" \
    ip "192.168.1.100" \
    user_agent "Mozilla/5.0..." \
    status "success"

XADD audit:actions * \
    timestamp "2024-01-15T10:31:00Z" \
    user_id "1001" \
    action "document.view" \
    resource_type "document" \
    resource_id "doc-123" \
    status "success"

XADD audit:actions * \
    timestamp "2024-01-15T10:32:00Z" \
    user_id "1001" \
    action "document.delete" \
    resource_type "document" \
    resource_id "doc-123" \
    status "denied" \
    reason "insufficient_permissions"
```

## Audit Logger Implementation

```python
from datetime import datetime
import json

class AuditLogger:
    def __init__(self, redis_client, stream_key='audit:actions'):
        self.r = redis_client
        self.stream_key = stream_key
    
    def log(self, user_id, action, status='success', **context):
        entry = {
            'timestamp': datetime.utcnow().isoformat(),
            'user_id': str(user_id),
            'action': action,
            'status': status,
            **{k: str(v) for k, v in context.items()}
        }
        
        # Auto-trim to last 1 million entries
        return self.r.xadd(
            self.stream_key,
            entry,
            maxlen=1000000,
            approximate=True
        )
    
    def get_user_actions(self, user_id, count=100):
        """Get recent actions by a user"""
        # Note: This requires scanning; consider per-user streams for efficiency
        all_entries = self.r.xrevrange(self.stream_key, '+', '-', count=count * 10)
        return [
            {'id': e[0], **e[1]}
            for e in all_entries
            if e[1].get('user_id') == str(user_id)
        ][:count]
    
    def get_actions_by_resource(self, resource_type, resource_id, count=100):
        """Get actions on a specific resource"""
        all_entries = self.r.xrevrange(self.stream_key, '+', '-', count=count * 10)
        return [
            {'id': e[0], **e[1]}
            for e in all_entries
            if e[1].get('resource_type') == resource_type
            and e[1].get('resource_id') == str(resource_id)
        ][:count]
    
    def get_recent_failures(self, count=50):
        """Get recent failed actions"""
        all_entries = self.r.xrevrange(self.stream_key, '+', '-', count=count * 5)
        return [
            {'id': e[0], **e[1]}
            for e in all_entries
            if e[1].get('status') in ('denied', 'error', 'failure')
        ][:count]
```

## Multi-Stream Architecture

For high-volume systems, use multiple streams:

```python
class ScalableAuditLogger:
    def __init__(self, redis_client):
        self.r = redis_client
    
    def log(self, user_id, action, **context):
        entry = {
            'timestamp': datetime.utcnow().isoformat(),
            'user_id': str(user_id),
            'action': action,
            **context
        }
        
        # Main stream (all events)
        self.r.xadd('audit:all', entry, maxlen=1000000, approximate=True)
        
        # Per-user stream
        self.r.xadd(f'audit:user:{user_id}', entry, maxlen=10000, approximate=True)
        
        # Per-action stream
        action_type = action.split('.')[0]  # e.g., "user" from "user.login"
        self.r.xadd(f'audit:action:{action_type}', entry, maxlen=100000, approximate=True)
    
    def get_user_history(self, user_id, count=100):
        """Fast lookup for user-specific history"""
        return self.r.xrevrange(f'audit:user:{user_id}', '+', '-', count=count)
```

## Compliance and Retention

```python
import time

def archive_old_audit_logs(days_to_keep=90):
    """Archive and delete old audit logs"""
    cutoff_ms = int((time.time() - days_to_keep * 86400) * 1000)
    cutoff_id = f"{cutoff_ms}-0"
    
    # Get entries to archive
    old_entries = r.xrange('audit:all', '-', cutoff_id, count=1000)
    
    if old_entries:
        # Archive to cold storage (S3, etc.)
        archive_to_cold_storage(old_entries)
        
        # Delete from Redis
        r.xtrim('audit:all', minid=cutoff_id, approximate=True)

# Run daily
archive_old_audit_logs()
```

📖 [Audit Log Patterns](https://redis.io/docs/latest/develop/data-types/streams/)

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*