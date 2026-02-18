---
source_course: "redis-messaging"
source_lesson: "redis-messaging-dlq"
---

# Dead Letter Queues and Delivery Guarantees

Handle messages that repeatedly fail to process and understand delivery semantics.

## Dead Letter Queue (DLQ) Pattern

Messages that fail processing too many times should be moved aside:

```python
MAX_RETRIES = 3

def process_with_dlq(stream, group, consumer):
    while True:
        # Check pending messages first
        pending = r.xpending_range(
            stream, group,
            min='-', max='+',
            count=10,
            consumername=consumer
        )
        
        for entry in pending:
            msg_id = entry['message_id']
            delivery_count = entry['times_delivered']
            
            if delivery_count > MAX_RETRIES:
                # Move to Dead Letter Queue
                msg = r.xrange(stream, msg_id, msg_id)
                if msg:
                    # Store in DLQ with metadata
                    r.xadd(f'{stream}:dlq', {
                        'original_id': msg_id,
                        'original_stream': stream,
                        'delivery_count': str(delivery_count),
                        'failed_at': str(time.time()),
                        **msg[0][1]
                    })
                
                # Acknowledge to remove from PEL
                r.xack(stream, group, msg_id)
                print(f"Moved {msg_id} to DLQ after {delivery_count} attempts")
                continue
            
            # Try to reprocess
            try:
                msg = r.xrange(stream, msg_id, msg_id)
                if msg:
                    process_message(msg[0][1])
                    r.xack(stream, group, msg_id)
            except Exception as e:
                print(f"Retry {delivery_count} failed for {msg_id}: {e}")
```

## DLQ Monitoring and Recovery

```python
def get_dlq_stats(stream):
    """Get statistics about dead letters"""
    dlq_key = f'{stream}:dlq'
    return {
        'count': r.xlen(dlq_key),
        'oldest': r.xrange(dlq_key, '-', '+', count=1),
        'newest': r.xrevrange(dlq_key, '+', '-', count=1)
    }

def reprocess_dlq(stream, count=10):
    """Attempt to reprocess dead letters"""
    dlq_key = f'{stream}:dlq'
    messages = r.xrange(dlq_key, '-', '+', count=count)
    
    reprocessed = 0
    for msg_id, fields in messages:
        try:
            # Extract original data (exclude DLQ metadata)
            original_data = {k: v for k, v in fields.items() 
                          if k not in ['original_id', 'original_stream', 
                                       'delivery_count', 'failed_at']}
            
            process_message(original_data)
            r.xdel(dlq_key, msg_id)
            reprocessed += 1
        except Exception as e:
            print(f"Still failing: {msg_id}")
    
    return reprocessed
```

## Delivery Guarantees

### At-Least-Once Delivery

```python
def at_least_once_consumer(stream, group, consumer):
    """Messages may be processed multiple times"""
    while True:
        messages = r.xreadgroup(
            groupname=group,
            consumername=consumer,
            streams={stream: '>'},
            count=10,
            block=5000
        )
        
        for stream_name, entries in (messages or []):
            for msg_id, fields in entries:
                try:
                    process_message(fields)  # Must be idempotent!
                    r.xack(stream, group, msg_id)
                except Exception:
                    pass  # Will be redelivered
```

### Exactly-Once Processing (Application Level)

```python
def exactly_once_consumer(stream, group, consumer):
    """Use deduplication for exactly-once semantics"""
    while True:
        messages = r.xreadgroup(
            groupname=group,
            consumername=consumer,
            streams={stream: '>'},
            count=10,
            block=5000
        )
        
        for stream_name, entries in (messages or []):
            for msg_id, fields in entries:
                # Check if already processed
                if r.sismember('processed:messages', msg_id):
                    r.xack(stream, group, msg_id)
                    continue
                
                try:
                    # Process in transaction with dedup marker
                    pipe = r.pipeline()
                    process_message_pipelined(pipe, fields)
                    pipe.sadd('processed:messages', msg_id)
                    pipe.expire('processed:messages', 86400)  # 24h
                    pipe.execute()
                    
                    r.xack(stream, group, msg_id)
                except Exception:
                    pass
```

## Stream Trimming Strategies

```redis
# Keep last 10000 messages
XTRIM mystream MAXLEN ~ 10000

# Remove messages older than specific ID
XTRIM mystream MINID ~ 1704067200000-0

# Auto-trim on XADD
XADD mystream MAXLEN ~ 10000 * field value
```

📖 [Streams Tutorial](https://redis.io/docs/latest/develop/data-types/streams-tutorial/)

## Resources

- [Streams Tutorial](https://redis.io/docs/latest/develop/data-types/streams-tutorial/) — Complete Redis Streams tutorial

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*