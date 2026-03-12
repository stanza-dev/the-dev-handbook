---
source_course: "redis-clustering"
source_lesson: "redis-clustering-disaster-recovery"
---

# Disaster Recovery Planning

## Introduction

Disaster recovery planning ensures your Redis deployment can survive catastrophic failures. This involves defining acceptable data loss (RPO), acceptable downtime (RTO), and building procedures to meet those targets.

## Key Concepts

- **RPO (Recovery Point Objective)**: Maximum acceptable data loss, measured in time
- **RTO (Recovery Time Objective)**: Maximum acceptable downtime until service is restored
- **RDB Snapshot**: Point-in-time binary backup of the Redis dataset
- **AOF (Append Only File)**: Write-ahead log for point-in-time recovery

## Real World Context

Every production Redis deployment needs a documented DR plan. Without one, a datacenter failure means scrambling under pressure to figure out recovery procedures, likely resulting in data loss and extended downtime.

## Deep Dive

### RPO and RTO

| Strategy | RPO | RTO |
|----------|-----|-----|
| Active-Active | ~0 | ~0 |
| Sentinel failover | Seconds of data | 5-15 seconds |
| Manual failover | Minutes of data | Minutes to hours |
| Restore from backup | Hours of data | Hours |

### Backup Strategies

**RDB Snapshots for DR:**
```redis
BGSAVE
LASTSAVE
```

```bash
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
redis-cli BGSAVE
sleep 5
cp /var/lib/redis/dump.rdb /backup/redis/dump_$DATE.rdb
aws s3 cp /backup/redis/dump_$DATE.rdb s3://my-backups/redis/
```

**AOF for Point-in-Time Recovery:**
```redis
BGREWRITEAOF
```

### Planned Failover Procedure

```bash
# 1. Disable writes on primary
redis-cli CONFIG SET min-replicas-to-write 999

# 2. Wait for replicas to sync
redis-cli INFO replication
# Check slave_repl_offset matches master_repl_offset

# 3. Promote replica
redis-cli -h replica REPLICAOF NO ONE

# 4. Update DNS/clients to point to new primary

# 5. Rejoin old primary as replica (optional)
redis-cli -h old-primary REPLICAOF new-primary 6379
```

### Emergency Failover

When the primary is unreachable:
1. Verify primary is truly down
2. Find the replica with the highest replication offset
3. Promote that replica with `REPLICAOF NO ONE`
4. Point other replicas to the new primary
5. Update application configuration/DNS

### DR Testing

**Monthly tests:**
- Restore from backup to test environment
- Verify data integrity
- Time the restoration process

**Quarterly tests:**
- Simulate datacenter failure
- Execute full failover procedure
- Measure actual RTO/RPO

## Common Pitfalls

1. **Never testing backup restoration** -- Backups that cannot be restored are useless. Regularly restore backups to a test environment to verify integrity.
2. **Assuming Sentinel handles everything** -- Sentinel handles single-node failures, not datacenter failures. Cross-region DR requires additional planning.

## Best Practices

1. **Document runbooks** -- Write step-by-step procedures for both planned and emergency failover. Engineers under stress need clear instructions.
2. **Automate backup rotation** -- Keep daily backups for a week, weekly for a month, and monthly for a year. Automate upload to off-site storage.

## Summary

- Define RPO and RTO targets before choosing a DR strategy
- Use RDB snapshots for periodic backups and AOF for point-in-time recovery
- Document and practice both planned and emergency failover procedures
- Test backup restoration monthly and full DR quarterly
- Store backups off-site (e.g., S3) for true disaster resilience

## Code Examples

**Automated Redis backup script with S3 upload**

```bash
#!/bin/bash
# Automated Redis backup script
DATE=$(date +%Y%m%d_%H%M%S)
redis-cli BGSAVE
sleep 5
cp /var/lib/redis/dump.rdb /backup/redis/dump_$DATE.rdb
aws s3 cp /backup/redis/dump_$DATE.rdb s3://my-backups/redis/
```


## Resources

- [Redis Persistence](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/) — Persistence and backup documentation

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*