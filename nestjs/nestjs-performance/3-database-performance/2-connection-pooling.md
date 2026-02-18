---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-connection-pooling"
---

# Connection Pooling

## Introduction

Database connections are expensive to create. Connection pooling maintains a pool of reusable connections, eliminating connection overhead and improving throughput. Proper pool configuration is critical for high-traffic applications.

## Key Concepts

- **Connection Pool**: Pre-established database connections
- **Pool Size**: Number of connections to maintain
- **Connection Lifecycle**: Acquire, use, release pattern
- **Pool Exhaustion**: When no connections are available

## Real World Context

Connection pooling prevents:
- Connection creation overhead on every request
- Database connection limits being hit
- Requests waiting for connections
- Connection leaks from unclosed connections

## Deep Dive

### TypeORM Pool Configuration

```typescript
TypeOrmModule.forRoot({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  database: 'mydb',
  // Connection pool settings
  extra: {
    max: 20,              // Maximum connections
    min: 5,               // Minimum connections
    idleTimeoutMillis: 30000,  // Close idle connections after 30s
    connectionTimeoutMillis: 2000,  // Wait 2s for connection
  },
});
```

### Pool Size Calculation

```typescript
// Rule of thumb: connections = ((core_count * 2) + effective_spindle_count)
// For SSD: connections ≈ core_count * 2
// For most apps: 10-20 connections is sufficient

const poolConfig = {
  max: Math.min(process.env.DB_POOL_MAX || 20, 100),
  min: Math.max(process.env.DB_POOL_MIN || 2, 1),
};
```

### Connection Health Checks

```typescript
import { Injectable, OnModuleInit } from '@nestjs/common';
import { DataSource } from 'typeorm';

@Injectable()
export class DatabaseHealthService implements OnModuleInit {
  constructor(private dataSource: DataSource) {}

  async onModuleInit() {
    // Verify connection on startup
    try {
      await this.dataSource.query('SELECT 1');
      console.log('Database connection verified');
    } catch (error) {
      console.error('Database connection failed', error);
      process.exit(1);
    }
  }

  async checkHealth() {
    try {
      await this.dataSource.query('SELECT 1');
      return { status: 'healthy' };
    } catch (error) {
      return { status: 'unhealthy', error: error.message };
    }
  }
}
```

### Monitoring Pool Usage

```typescript
import { DataSource } from 'typeorm';

@Injectable()
export class PoolMonitorService {
  constructor(private dataSource: DataSource) {}

  getPoolStats() {
    const pool = (this.dataSource.driver as any).pool;
    return {
      total: pool.totalCount,
      idle: pool.idleCount,
      waiting: pool.waitingCount,
    };
  }

  @Cron('*/30 * * * * *')  // Every 30 seconds
  logPoolStats() {
    const stats = this.getPoolStats();
    if (stats.waiting > 0) {
      this.logger.warn('Pool exhaustion warning', stats);
    }
  }
}
```

### Transaction Management

```typescript
@Injectable()
export class OrdersService {
  constructor(private dataSource: DataSource) {}

  async createOrder(dto: CreateOrderDto): Promise<Order> {
    // Use transaction to ensure connection is released
    return this.dataSource.transaction(async (manager) => {
      const order = manager.create(Order, dto);
      await manager.save(order);

      for (const item of dto.items) {
        await manager.save(OrderItem, { ...item, orderId: order.id });
      }

      return order;
    });
  }
}
```

### Connection Release

```typescript
// TypeORM handles this automatically with repository methods
// Manual query builder needs attention

// BAD: Potentially leaks connection
const runner = this.dataSource.createQueryRunner();
await runner.connect();
await runner.query('SELECT * FROM users');
// Missing: await runner.release();

// GOOD: Always release
const runner = this.dataSource.createQueryRunner();
try {
  await runner.connect();
  await runner.query('SELECT * FROM users');
} finally {
  await runner.release();
}
```

## Common Pitfalls

1. **Pool too small**: Requests queue waiting for connections.
2. **Pool too large**: Wastes database resources.
3. **Connection leaks**: Not releasing connections after use.

## Best Practices

- Start with pool size 10-20, adjust based on monitoring
- Always release connections (use transactions or repositories)
- Monitor pool usage in production
- Set appropriate timeouts
- Health check database on startup

## Summary

Connection pooling reuses database connections for efficiency. Configure pool size based on workload, monitor usage, and ensure connections are released. Use transactions for multi-query operations and health checks for reliability.

## Resources

- [Database](https://docs.nestjs.com/techniques/database) — Connection management

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*