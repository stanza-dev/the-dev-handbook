---
source_course: "redis-messaging"
source_lesson: "redis-messaging-audit-logs"
---

# Audit Logs with Redis Streams

## Introduction

Audit logs track every action taken in a system — who did what, when, and with what result. Redis Streams are an excellent audit log backend due to their append-only nature and time-ordered IDs.

## Key Concepts

- **Append-only log**: stream entries cannot be modified after creation, preserving integrity
- **Time-ordered IDs**: the millisecond-based stream ID enables precise time-range queries without a separate timestamp field
- **Two-tier retention**: hot tier (Redis, recent events) + cold tier (S3/BigQuery, long-term compliance)
- **Domain-separated streams**: `audit:auth`, `audit:data`, `audit:admin` — separate streams per category for different retention and access policies

## Real World Context

- GDPR compliance: log every data access to `audit:data` with userId and resource
- Security: log all auth events including failed login attempts
- Billing: track every API call for usage-based billing

## Deep Dive

```bash
# Log every significant action
XADD audit '*' userId 42 action document:delete resourceId doc-99 ip 192.168.1.1 result success

# Events in a time window (e.g., today from 9am to 5pm)
XRANGE audit 1691226000000 1691254800000

# Most recent 20 events
XREVRANGE audit + - COUNT 20

# Trim to 90 days retention
XTRIM audit:auth MINID ~ 1683454967000
```

Separate streams per domain:

```bash
XADD audit:auth '*' ...    # login/logout events
XADD audit:data '*' ...    # data access events
XADD audit:admin '*' ...   # admin actions
```

## Common Pitfalls

- **Not separating audit domains**: mixing auth, data access, and admin events in one stream makes retention policies impossible to apply per category
- **Storing full object payloads**: large entries degrade performance; store only the delta or a reference ID
- **Relying solely on Redis for long-term compliance**: Redis is not designed for multi-year retention; export to cold storage

## Best Practices

- Use domain-separated streams (`audit:auth`, `audit:data`) for per-category retention and access control
- Set up a nightly export job to move aged-out entries to S3 or BigQuery before XTRIM removes them
- Always include `userId`, `action`, `resourceId`, and `result` in every audit entry

## Summary

Redis Streams are a natural audit log: append-only, time-ordered, queryable by time range. Use XRANGE with timestamp-based IDs for time-window queries. Implement a two-tier strategy (hot Redis + cold object store) for long-term compliance retention.

## Code Examples

**Audit log implementation**

```bash
# Write audit events
XADD audit:auth '*' userId 42 action login ip 203.0.113.5 result success
XADD audit:data '*' userId 42 action read resourceId doc-99 classification PII

# Query: events in last hour
# (current timestamp minus 3600 seconds in ms)
XRANGE audit:auth 1691230967000 + 

# Query: last 50 events reverse chronological
XREVRANGE audit:data + - COUNT 50

# Trim to 90 days retention
XTRIM audit:auth MINID ~ 1683454967000
```


## Resources

- [Redis Streams](https://redis.io/docs/latest/develop/data-types/streams/) — Official Redis Streams documentation

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*