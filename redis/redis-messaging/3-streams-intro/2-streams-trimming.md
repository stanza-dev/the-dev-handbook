---
source_course: "redis-messaging"
source_lesson: "redis-messaging-streams-trimming"
---

# Stream Management and Trimming

Streams grow indefinitely unless trimmed. Learn how to manage stream size effectively.

## XTRIM: Trimming Streams

### By Maximum Length

```redis
# Keep only the latest 1000 entries
XTRIM orders MAXLEN 1000
# Returns: number of entries removed

# Approximate trimming (faster, ~100 entry precision)
XTRIM orders MAXLEN ~ 1000
```

### By Minimum ID

```redis
# Remove entries older than a specific ID
XTRIM orders MINID 1704067200000-0

# Approximate (faster)
XTRIM orders MINID ~ 1704067200000-0
```

## Auto-Trimming with XADD

```redis
# Add and trim in one command
XADD orders MAXLEN ~ 1000 * customer alice total 99.99

# Using MINID
XADD orders MINID ~ 1704000000000-0 * customer bob total 50.00
```

This is more efficient than separate XADD + XTRIM.

## XDEL: Deleting Specific Entries

```redis
# Delete specific entries by ID
XDEL orders 1704067200001-0
# Returns: (integer) 1

# Delete multiple
XDEL orders 1704067200001-0 1704067200002-0
```

**Note**: XDEL marks entries as deleted but doesn't reclaim memory until the stream is compacted.

## XINFO: Stream Information

### Stream Details

```redis
XINFO STREAM orders
# Returns:
# 1) "length"
# 2) (integer) 1000
# 3) "first-entry"
# 4) 1) "1704067200000-0"
#    2) 1) "customer"
#       2) "alice"
# 5) "last-entry"
# 6) ...
```

### Consumer Groups Info

```redis
# List all consumer groups
XINFO GROUPS orders

# Details about consumers in a group
XINFO CONSUMERS orders mygroup
```

## Retention Strategies

### Time-Based Retention

```python
import time

def trim_by_age(stream_key, max_age_seconds):
    """Remove entries older than max_age_seconds"""
    min_timestamp = int((time.time() - max_age_seconds) * 1000)
    min_id = f"{min_timestamp}-0"
    r.xtrim(stream_key, minid=min_id, approximate=True)

# Keep only last 24 hours
trim_by_age('events', 86400)
```

### Size-Based Retention

```python
def trim_by_size(stream_key, max_entries):
    """Keep only the latest max_entries"""
    r.xtrim(stream_key, maxlen=max_entries, approximate=True)

# Keep last 10000 entries
trim_by_size('events', 10000)
```

### Scheduled Trimming

```python
import schedule

def daily_trim():
    streams = ['events', 'orders', 'logs']
    for stream in streams:
        trim_by_age(stream, 7 * 86400)  # 7 days
        print(f"Trimmed {stream}")

schedule.every().day.at("03:00").do(daily_trim)
```

## Memory Considerations

```redis
# Check stream memory usage
MEMORY USAGE orders
# Returns: bytes used

# Check overall memory
INFO memory
```

**Best Practices:**
- Use approximate trimming (`~`) for better performance
- Trim during low-traffic periods
- Monitor memory usage regularly
- Set appropriate MAXLEN with XADD for automatic management

📖 [XTRIM Command](https://redis.io/docs/latest/commands/xtrim/)

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*