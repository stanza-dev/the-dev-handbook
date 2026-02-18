---
source_course: "redis-messaging"
source_lesson: "redis-messaging-xclaim"
---

# Message Claiming with XCLAIM

When a consumer fails, its pending messages need to be recovered. XCLAIM allows other consumers to take ownership of orphaned messages.

## The Pending Entries List (PEL)

Every message read but not acknowledged is tracked:

```redis
# View pending messages details
XPENDING orders orderprocessors - + 10 consumer-1

# Returns:
# 1) 1) "1704067200000-0"     # Entry ID
#    2) "consumer-1"          # Current owner
#    3) (integer) 300000      # Idle time (ms)
#    4) (integer) 3           # Delivery count
```

## XCLAIM: Take Over Messages

```redis
# Claim messages idle for more than 5 minutes
XCLAIM orders orderprocessors consumer-2 300000 1704067200000-0
# 300000 = minimum idle time in ms (5 minutes)

# Returns the claimed message:
# 1) 1) "1704067200000-0"
#    2) 1) "customer"
#       2) "alice"
#       3) "total"
#       4) "99.99"
```

### XCLAIM Options

```redis
# Just get IDs without message data (faster)
XCLAIM orders orderprocessors consumer-2 300000 1704067200000-0 JUSTID

# Reset delivery count
XCLAIM orders orderprocessors consumer-2 300000 1704067200000-0 RETRYCOUNT 0

# Force claim regardless of idle time
XCLAIM orders orderprocessors consumer-2 0 1704067200000-0 FORCE
```

## XAUTOCLAIM: Automatic Claiming

Scans and claims idle messages automatically:

```redis
# Claim messages idle > 5 minutes, return up to 10
XAUTOCLAIM orders orderprocessors consumer-2 300000 0-0 COUNT 10

# Returns:
# 1) "1704067200003-0"        # Next ID for pagination
# 2) 1) 1) "1704067200000-0"  # Claimed messages
#       2) 1) "customer"
#          2) "alice"
# 3) 1) "1704067200001-0"     # IDs that no longer exist
```

### Implementing Automatic Recovery

```python
import redis
import time

r = redis.Redis(decode_responses=True)

def recover_abandoned_messages(stream, group, consumer, idle_ms=300000):
    """Claim and process messages from failed consumers"""
    start_id = '0-0'
    
    while True:
        result = r.xautoclaim(
            stream, group, consumer,
            min_idle_time=idle_ms,
            start_id=start_id,
            count=100
        )
        
        next_id, claimed_messages, deleted_ids = result
        
        if not claimed_messages:
            break
        
        for msg_id, fields in claimed_messages:
            try:
                process_message(fields)
                r.xack(stream, group, msg_id)
                print(f"Recovered and processed: {msg_id}")
            except Exception as e:
                print(f"Failed to process {msg_id}: {e}")
        
        if next_id == '0-0':
            break
        start_id = next_id

# Run periodically
while True:
    recover_abandoned_messages('orders', 'orderprocessors', 'recovery-worker')
    time.sleep(60)  # Check every minute
```

📖 [XCLAIM Command](https://redis.io/docs/latest/commands/xclaim/)

## Resources

- [XCLAIM Command](https://redis.io/docs/latest/commands/xclaim/) — XCLAIM command reference

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*