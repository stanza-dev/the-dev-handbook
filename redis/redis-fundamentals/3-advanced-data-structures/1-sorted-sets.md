---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-sorted-sets"
---

# Sorted Sets: Rankings and Scores

## Introduction

Sorted Sets combine the uniqueness of Sets with a floating-point score for each member, keeping elements automatically sorted by score. They are the definitive data structure for leaderboards, rankings, priority queues, and time-series indices.

## Key Concepts

- **Score-based ordering**: Each member has a numeric score; Redis maintains sort order automatically.
- **Rank queries**: ZRANK and ZREVRANK return a member's position in the sorted order.
- **Range queries**: ZRANGE supports rank-based, score-based, and lexicographic ranges.
- **ZINCRBY**: Atomically increment a member's score — perfect for accumulating points.

## Real World Context

Every game leaderboard, trending content feed, and priority task queue in production likely runs on sorted sets. Time-series indices store events with timestamps as scores for efficient range queries. Rate limiters use sorted sets for sliding window algorithms.

## Deep Dive

Sorted Sets combine the uniqueness of Sets with a score for each member, keeping elements automatically sorted by score. Perfect for leaderboards, priority queues, and time-series data.

## How Sorted Sets Work

```
┌────────────────────────────────────────┐
│  Member      │  Score                  │
├──────────────┼─────────────────────────┤
│  "alice"     │  100                    │
│  "bob"       │  250                    │
│  "charlie"   │  175                    │
└────────────────────────────────────────┘
         ↓ Automatically sorted by score
┌────────────────────────────────────────┐
│  "alice" (100) → "charlie" (175) → "bob" (250)
└────────────────────────────────────────┘
```

## Basic Commands

### ZADD - Adding Members

```redis
# Add single member
ZADD leaderboard 100 "alice"
# Returns: 1

# Add multiple members
ZADD leaderboard 250 "bob" 175 "charlie" 300 "diana"
# Returns: 3

# Update existing score
ZADD leaderboard 150 "alice"   # alice now has 150
# Returns: 0 (no new members added)

# Only add if NOT exists
ZADD leaderboard NX 500 "alice"   # Ignored, alice exists

# Only update if EXISTS
ZADD leaderboard XX 200 "alice"   # Updates alice to 200
```

### ZRANGE - Getting Members

```redis
# Get by rank (position), lowest to highest
ZRANGE leaderboard 0 -1
# Returns: ["alice", "charlie", "bob", "diana"]

# Get with scores
ZRANGE leaderboard 0 -1 WITHSCORES
# Returns: ["alice", "150", "charlie", "175", "bob", "250", "diana", "300"]

# Highest to lowest (REV)
ZRANGE leaderboard 0 2 REV
# Returns: ["diana", "bob", "charlie"] (top 3)
```

### Score and Rank Queries

```redis
# Get score of member
ZSCORE leaderboard "bob"
# Returns: "250"

# Get rank (0-indexed, lowest score = 0)
ZRANK leaderboard "bob"
# Returns: 2 (third position)

# Reverse rank (highest score = 0)
ZREVRANK leaderboard "bob"
# Returns: 1 (second from top)

# Count members
ZCARD leaderboard
# Returns: 4
```

## Score Operations

```redis
# Increment score
ZINCRBY leaderboard 50 "alice"
# Returns: "200" (new score)

# Count members in score range
ZCOUNT leaderboard 100 200
# Returns: count of members with scores between 100 and 200

# Get members by score range
ZRANGE leaderboard 100 200 BYSCORE
# Returns: members with scores 100-200

ZRANGE leaderboard 100 200 BYSCORE LIMIT 0 5
# Returns: first 5 members in that range
```

## Removing Members

```redis
# Remove specific members
ZREM leaderboard "alice"
# Returns: 1

# Remove by rank range
ZREMRANGEBYRANK leaderboard 0 1
# Removes lowest 2 scores

# Remove by score range
ZREMRANGEBYSCORE leaderboard 0 100
# Removes all with scores 0-100
```

## Practical Example: Game Leaderboard

```redis
# Players complete a game
ZADD game:scores 1500 "player:1"
ZADD game:scores 2200 "player:2"
ZADD game:scores 1800 "player:3"
ZADD game:scores 3100 "player:4"
ZADD game:scores 2900 "player:5"

# Get top 3 players
ZRANGE game:scores 0 2 REV WITHSCORES
# Returns: ["player:4", "3100", "player:5", "2900", "player:2", "2200"]

# Player improves their score
ZADD game:scores GT 2500 "player:3"   # Only update if new score is Greater Than

# Get player's rank
ZREVRANK game:scores "player:3"
# Returns their position (0 = first place)

# Get players around a specific rank (for "players near you")
ZRANGE game:scores 2 4 REV WITHSCORES
# Returns ranks 3-5
```

## Time-Series with Sorted Sets

```redis
# Store events with timestamp as score
ZADD events 1704067200 "event:1"
ZADD events 1704153600 "event:2"
ZADD events 1704240000 "event:3"

# Get events in time range
ZRANGE events 1704067200 1704200000 BYSCORE
```

📖 [Sorted Set Commands](https://redis.io/docs/latest/commands/?group=sorted-set)

## Common Pitfalls

1. **Confusing ZRANK with ZREVRANK** — ZRANK returns rank by ascending score (lowest = 0). For "top scores" use ZREVRANK where highest = 0.
2. **Using ZRANGEBYSCORE instead of ZRANGE BYSCORE** — The old ZRANGEBYSCORE command still works but ZRANGE with BYSCORE is the modern, unified syntax.

## Best Practices

1. **Use ZADD GT for high-score tables** — GT only updates a score if the new value is greater, preventing accidental score decreases.
2. **Combine ZRANGE REV with LIMIT for pagination** — Fetch top-N results efficiently without loading the entire set.

## Summary

- Sorted Sets maintain members in score order automatically.
- ZADD adds members with scores; ZINCRBY updates scores atomically.
- ZRANGE with REV and WITHSCORES is the primary query command.
- ZREVRANK returns a member's position from the top (highest = 0).
- Use GT flag with ZADD to only allow score increases.

## Code Examples

**Sorted Set leaderboard — ZADD adds members with scores, ZRANGE REV returns highest first, ZINCRBY updates scores**

```bash
# Create a leaderboard
ZADD leaderboard 1500 "alice" 2200 "bob" 3100 "charlie"

# Get top 3 (highest scores first)
ZRANGE leaderboard 0 2 REV WITHSCORES
# 1) "charlie"  2) "3100"
# 3) "bob"      4) "2200"
# 5) "alice"    6) "1500"

# Increment a player's score
ZINCRBY leaderboard 500 "alice"
# "2000"
```


## Resources

- [Redis Sorted Sets](https://redis.io/docs/latest/develop/data-types/sorted-sets/) — Complete guide to Redis Sorted Sets

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*