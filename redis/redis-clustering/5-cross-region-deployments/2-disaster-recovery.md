---
source_course: "redis-clustering"
source_lesson: "redis-clustering-disaster-recovery"
---

# Disaster Recovery Planning

Prepare for the worst with proper DR strategies and testing.

## RPO and RTO Concepts

```
┌─────────────────────────────────────────────────────────────┐
│  Recovery Point Objective (RPO)                             │
│  How much data can you afford to lose?                      │
│  • RPO = 0: No data loss (sync replication)                │
│  • RPO = 1 minute: Up to 1 minute of data loss             │
├─────────────────────────────────────────────────────────────┤
│  Recovery Time Objective (RTO)                              │
│  How long can you be down?                                  │
│  • RTO = 0: Instant failover (active-active)               │
│  • RTO = 5 minutes: Automatic Sentinel failover            │
│  • RTO = 1 hour: Manual intervention required              │
└─────────────────────────────────────────────────────────────┘
```

## Backup Strategies

### RDB Snapshots for DR

```redis
# Force immediate snapshot
BGSAVE

# Check last save time
LASTSAVE
```

```bash
# Backup script
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
redis-cli BGSAVE
sleep 5
cp /var/lib/redis/dump.rdb /backup/redis/dump_$DATE.rdb
aws s3 cp /backup/redis/dump_$DATE.rdb s3://my-backups/redis/
```

### AOF for Point-in-Time Recovery

```redis
# Rewrite AOF to compact it
BGREWRITEAOF
```

```bash
# Backup AOF file
cp /var/lib/redis/appendonly.aof /backup/redis/aof_$DATE.aof
```

## Failover Procedures

### Planned Failover (Maintenance)

```bash
# 1. Disable writes on primary
redis-cli CONFIG SET min-replicas-to-write 999

# 2. Wait for replicas to sync
redis-cli INFO replication
# Check slave_repl_offset matches master_repl_offset

# 3. Promote replica
redis-cli -h replica REPLICAOF NO ONE

# 4. Update DNS/clients to point to new primary

# 5. Perform maintenance on old primary

# 6. Rejoin as replica (optional)
redis-cli -h old-primary REPLICAOF new-primary 6379
```

### Emergency Failover

```python
def emergency_failover(old_primary, new_primary):
    """Manual failover when primary is unreachable"""
    
    # 1. Verify primary is truly down
    try:
        old_primary.ping()
        print("Primary is up! Aborting.")
        return False
    except:
        print("Primary confirmed down")
    
    # 2. Find best replica (most up-to-date)
    best_replica = None
    best_offset = -1
    
    for replica in replicas:
        try:
            info = replica.info('replication')
            offset = info.get('slave_repl_offset', 0)
            if offset > best_offset:
                best_offset = offset
                best_replica = replica
        except:
            continue
    
    # 3. Promote best replica
    best_replica.execute_command('REPLICAOF', 'NO', 'ONE')
    
    # 4. Point other replicas to new primary
    for replica in replicas:
        if replica != best_replica:
            replica.execute_command('REPLICAOF', new_primary_host, 6379)
    
    # 5. Update application configuration
    update_dns(new_primary_host)
    
    return True
```

## DR Testing

**Monthly tests:**
- Restore from backup to test environment
- Verify data integrity
- Time the restoration process

**Quarterly tests:**
- Simulate datacenter failure
- Execute full failover procedure
- Measure actual RTO/RPO

📖 [Redis Persistence](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/)

## Resources

- [Redis Persistence](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/) — Persistence and backup documentation

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*