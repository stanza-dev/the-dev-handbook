---
source_course: "redis-messaging"
source_lesson: "redis-messaging-failure-handling"
---

# Handling Consumer Failures

When consumers fail, their pending messages need to be recovered. Redis provides mechanisms for this.

## The Pending Entries List (PEL)

Every message read by a consumer (but not acknowledged) is tracked in the PEL:

```redis
# View pending messages
XPENDING orders orderprocessors - + 10 worker-1

# Returns:
# 1) 1) "1704067200000-0"     # Entry ID
#    2) "worker-1"            # Consumer
#    3) (integer) 60000       # Time since delivery (ms)
#    4) (integer) 1           # Delivery count
```

## Re-Reading Pending Messages

A consumer can re-read its own pending messages:

```redis
# Read pending messages (use 0 instead of >)
XREADGROUP GROUP orderprocessors worker-1 \
    COUNT 10 \
    STREAMS orders 0

# 0 = start from consumer's first pending message
```

## XCLAIM: Claiming Messages from Failed Consumers

If a consumer crashes, another can claim its messages:

```redis
# Claim messages idle for more than 60 seconds
XCLAIM orders orderprocessors worker-2 60000 \
    1704067200000-0 1704067200001-0

# 60000 = minimum idle time (ms)
# Only claims messages from consumers that have been idle
```

## XAUTOCLAIM: Automatic Claiming

Automatically claim and return idle messages:

```redis
# Claim messages idle > 60 seconds, return up to 10
XAUTOCLAIM orders orderprocessors worker-2 60000 0-0 COUNT 10

# Returns:
# 1) "1704067200002-0"     # Next ID for pagination
# 2) 1) 1) "1704067200000-0"  # Claimed entries
#       2) 1) "customer"
#          2) "alice"
```

## Dead Letter Queue Pattern

For messages that repeatedly fail:

```python
MAX_RETRIES = 3

def process_with_dlq(consumer_name):
    while True:
        # First, check for pending messages to retry
        pending = r.xpending_range(
            'orders', 'orderprocessors',
            min='-', max='+', count=10,
            consumername=consumer_name
        )
        
        for entry in pending:
            entry_id = entry['message_id']
            delivery_count = entry['times_delivered']
            
            if delivery_count > MAX_RETRIES:
                # Move to dead letter queue
                msg = r.xrange('orders', entry_id, entry_id)
                if msg:
                    r.xadd('orders:dlq', {'original_id': entry_id, **msg[0][1]})
                r.xack('orders', 'orderprocessors', entry_id)
                print(f"Moved {entry_id} to DLQ after {delivery_count} attempts")
        
        # Then process new messages
        messages = r.xreadgroup(
            groupname='orderprocessors',
            consumername=consumer_name,
            streams={'orders': '>'},
            count=10,
            block=5000
        )
        
        # ... process messages ...
```

## Consumer Health Check

```python
def claim_abandoned_messages(idle_threshold_ms=300000):
    """Claim messages from consumers idle > 5 minutes"""
    while True:
        result = r.xautoclaim(
            'orders',
            'orderprocessors',
            'recovery-worker',
            idle_threshold_ms,
            '0-0',
            count=100
        )
        
        next_id, claimed = result[0], result[1]
        
        if not claimed:
            break
        
        for entry_id, fields in claimed:
            # Reprocess claimed messages
            try:
                process_order(fields)
                r.xack('orders', 'orderprocessors', entry_id)
            except Exception:
                pass  # Will be claimed again if still failing
```

## Best Practices

1. **Set reasonable idle thresholds**: 1-5 minutes typically
2. **Implement retry limits**: Move to DLQ after N failures
3. **Monitor PEL size**: Alert if pending messages grow
4. **Use consumer heartbeats**: Keep consumers "alive" in the system
5. **Log delivery counts**: Track problematic messages

📖 [XCLAIM Command](https://redis.io/docs/latest/commands/xclaim/)

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*