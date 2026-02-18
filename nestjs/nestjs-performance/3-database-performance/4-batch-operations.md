---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-batch-operations"
---

# Batch Operations

## Introduction

Processing items one at a time is inefficient. Batch operations group multiple operations into single database calls, dramatically improving throughput for bulk data processing.

## Key Concepts

- **Bulk Insert**: Insert multiple rows in one query
- **Bulk Update**: Update multiple rows efficiently
- **Batch Processing**: Process data in chunks
- **Upsert**: Insert or update in one operation

## Real World Context

Batch operations handle:
- Data imports and migrations
- Bulk status updates
- Report generation
- Scheduled data processing

## Deep Dive

### Bulk Insert

```typescript
@Injectable()
export class UsersService {
  // BAD: One query per user
  async createManyBad(dtos: CreateUserDto[]): Promise<User[]> {
    const users: User[] = [];
    for (const dto of dtos) {
      const user = await this.usersRepo.save(dto);
      users.push(user);
    }
    return users;
  }

  // GOOD: Single bulk insert
  async createMany(dtos: CreateUserDto[]): Promise<User[]> {
    const users = this.usersRepo.create(dtos);
    return this.usersRepo.save(users);  // Single INSERT
  }

  // BETTER: Using insert for maximum performance (no returned entities)
  async bulkInsert(dtos: CreateUserDto[]): Promise<void> {
    await this.usersRepo.insert(dtos);  // Raw INSERT
  }
}
```

### Bulk Update

```typescript
@Injectable()
export class OrdersService {
  // Update multiple orders in one query
  async markAsShipped(orderIds: string[]): Promise<void> {
    await this.ordersRepo
      .createQueryBuilder()
      .update(Order)
      .set({ status: 'shipped', shippedAt: new Date() })
      .where('id IN (:...ids)', { ids: orderIds })
      .execute();
  }

  // Conditional bulk update
  async updatePrices(updates: { id: string; price: number }[]): Promise<void> {
    await this.dataSource.transaction(async (manager) => {
      for (const { id, price } of updates) {
        await manager.update(Product, id, { price });
      }
    });
  }
}
```

### Chunked Processing

```typescript
@Injectable()
export class DataProcessingService {
  private readonly CHUNK_SIZE = 1000;

  async processAllUsers(): Promise<void> {
    let offset = 0;
    let hasMore = true;

    while (hasMore) {
      const users = await this.usersRepo.find({
        skip: offset,
        take: this.CHUNK_SIZE,
        order: { id: 'ASC' },
      });

      if (users.length === 0) {
        hasMore = false;
        break;
      }

      await this.processChunk(users);
      offset += users.length;

      this.logger.log(`Processed ${offset} users`);
    }
  }

  private async processChunk(users: User[]): Promise<void> {
    // Process batch of users
    await Promise.all(
      users.map(user => this.updateUserMetrics(user)),
    );
  }
}
```

### Upsert Operations

```typescript
@Injectable()
export class ProductSyncService {
  async syncProducts(products: ExternalProduct[]): Promise<void> {
    // Upsert: insert or update on conflict
    await this.productsRepo
      .createQueryBuilder()
      .insert()
      .into(Product)
      .values(products.map(p => ({
        externalId: p.id,
        name: p.name,
        price: p.price,
      })))
      .orUpdate(['name', 'price'], ['externalId'])  // Update on conflict
      .execute();
  }
}
```

### Stream Processing for Large Datasets

```typescript
@Injectable()
export class ExportService {
  async exportToCSV(res: Response): Promise<void> {
    res.setHeader('Content-Type', 'text/csv');
    res.setHeader('Content-Disposition', 'attachment; filename=export.csv');

    const stream = await this.usersRepo
      .createQueryBuilder('user')
      .stream();

    stream.on('data', (row) => {
      res.write(`${row.user_id},${row.user_name},${row.user_email}\n`);
    });

    stream.on('end', () => {
      res.end();
    });
  }
}
```

## Common Pitfalls

1. **Memory exhaustion**: Load data in chunks, not all at once.
2. **Transaction timeouts**: Long batches may timeout.
3. **No progress tracking**: Users don't know if it's working.

## Best Practices

- Use bulk operations instead of loops
- Process large datasets in chunks
- Use streams for exports
- Implement progress tracking for long operations
- Use transactions for atomic batch operations

## Summary

Batch operations use bulk insert/update, chunked processing, and streams for efficiency. Process large datasets in manageable chunks, use upsert for sync operations, and stream data for exports.

## Resources

- [Database](https://docs.nestjs.com/techniques/database) — Database operations

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*