---
source_course: "redis-messaging"
source_lesson: "redis-messaging-consumer-groups"
---

# Understanding Consumer Groups

Consumer groups allow multiple consumers to cooperatively process messages from a stream, with each message delivered to only one consumer in the group.

## Why Consumer Groups?

```
┌─────────────────────────────────────────────────────────────┐
│  Without Consumer Groups:                                   │
│  Stream ──▶ Consumer A receives ALL messages               │
│        ──▶ Consumer B receives ALL messages (duplicate!)   │
├─────────────────────────────────────────────────────────────┤
│  With Consumer Groups:                                      │
│  Stream ──▶ Group ──▶ Consumer A receives message 1, 3, 5  │
│                  ──▶ Consumer B receives message 2, 4, 6   │
│  (Messages are distributed, not duplicated)                │
└─────────────────────────────────────────────────────────────┘
```

## Creating Consumer Groups

```redis
# Create group starting from beginning of stream
XGROUP CREATE orders orderprocessors 0
# 0 = start from first entry

# Create group for only new messages
XGROUP CREATE orders orderprocessors $ MKSTREAM
# $ = only new messages
# MKSTREAM = create stream if doesn't exist

# Create group from specific ID
XGROUP CREATE orders orderprocessors 1704067200000-0
```

## XREADGROUP: Reading as Consumer

```redis
# Read as consumer "worker-1" in group "orderprocessors"
XREADGROUP GROUP orderprocessors worker-1 \
    COUNT 10 \
    STREAMS orders >

# > means: give me messages never delivered to any consumer
```

The `>` special ID:
- Delivers only new, unprocessed messages
- Each message goes to ONE consumer only
- Messages are tracked in Pending Entries List (PEL)

## Message Acknowledgment

```redis
# After successfully processing, acknowledge the message
XACK orders orderprocessors 1704067200000-0

# Acknowledge multiple messages
XACK orders orderprocessors 1704067200000-0 1704067200001-0 1704067200002-0
```

**Unacknowledged messages remain pending** and can be:
- Re-read by the same consumer
- Claimed by another consumer (if the original consumer fails)

## Consumer Group Workflow

```python
import redis

r = redis.Redis(decode_responses=True)

def process_orders(consumer_name):
    group = 'orderprocessors'
    stream = 'orders'
    
    while True:
        # Read new messages
        messages = r.xreadgroup(
            groupname=group,
            consumername=consumer_name,
            streams={stream: '>'},
            count=10,
            block=5000  # Block for 5 seconds if no messages
        )
        
        if not messages:
            continue
        
        for stream_name, entries in messages:
            for entry_id, fields in entries:
                try:
                    # Process the order
                    process_order(fields)
                    
                    # Acknowledge on success
                    r.xack(stream, group, entry_id)
                    print(f"Processed: {entry_id}")
                except Exception as e:
                    # Don't acknowledge - will be retried
                    print(f"Failed to process {entry_id}: {e}")

def process_order(fields):
    # Business logic here
    customer = fields['customer']
    total = fields['total']
    print(f"Processing order for {customer}: ${total}")
```

## Running Multiple Consumers

```bash
# Terminal 1
python worker.py --name worker-1

# Terminal 2
python worker.py --name worker-2

# Terminal 3
python worker.py --name worker-3
```

Messages are automatically distributed among consumers!

## Checking Pending Messages

```redis
# Get pending entries summary
XPENDING orders orderprocessors
# Returns:
# 1) (integer) 5          # total pending
# 2) "1704067200000-0"    # smallest ID
# 3) "1704067200004-0"    # largest ID
# 4) 1) 1) "worker-1"     # consumers with pending
#       2) "3"
#    2) 1) "worker-2"
#       2) "2"

# Get detailed pending entries
XPENDING orders orderprocessors - + 10
```

📖 [Consumer Groups](https://redis.io/docs/latest/develop/data-types/streams/#consumer-groups)

## Resources

- [Streams Consumer Groups](https://redis.io/docs/latest/develop/data-types/streams/#consumer-groups) — Consumer groups documentation

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*