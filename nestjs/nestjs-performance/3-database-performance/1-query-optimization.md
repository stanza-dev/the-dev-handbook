---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-query-optimization"
---

# Query Optimization

## Introduction

Database queries are often the biggest performance bottleneck. Understanding query patterns, indexing strategies, and ORM pitfalls transforms slow APIs into responsive ones. A single optimized query can replace dozens of inefficient ones.

## Key Concepts

- **N+1 Problem**: Multiple queries where one would suffice
- **Eager Loading**: Load related data in fewer queries
- **Query Builder**: Fine-grained control over SQL generation
- **Indexing**: Speed up lookups with proper indexes

## Real World Context

Query optimization fixes:
- Slow dashboard loading (N+1 queries)
- Search timeouts (missing indexes)
- Memory exhaustion (loading too much data)
- Connection pool depletion (long-running queries)

## Deep Dive

### N+1 Problem

```typescript
// BAD: N+1 queries
const orders = await this.ordersRepo.find();
for (const order of orders) {
  const items = await this.orderItemsRepo.find({ where: { orderId: order.id } });
  order.items = items;
}
// 1 query for orders + N queries for items

// GOOD: Single query with join
const orders = await this.ordersRepo.find({
  relations: ['items'],
});
// 1 query with JOIN
```

### TypeORM Eager Loading

```typescript
// Entity with eager relation
@Entity()
export class Order {
  @OneToMany(() => OrderItem, item => item.order, { eager: true })
  items: OrderItem[];
}

// Or explicit loading
const orders = await this.ordersRepo.find({
  relations: ['items', 'customer', 'items.product'],
});
```

### Query Builder for Complex Queries

```typescript
const result = await this.ordersRepo
  .createQueryBuilder('order')
  .leftJoinAndSelect('order.items', 'item')
  .leftJoinAndSelect('item.product', 'product')
  .leftJoinAndSelect('order.customer', 'customer')
  .where('order.status = :status', { status: 'pending' })
  .andWhere('order.createdAt > :date', { date: lastWeek })
  .orderBy('order.createdAt', 'DESC')
  .take(20)
  .getMany();
```

### Select Specific Fields

```typescript
// BAD: Loading all columns
const users = await this.usersRepo.find();

// GOOD: Select only needed fields
const users = await this.usersRepo
  .createQueryBuilder('user')
  .select(['user.id', 'user.name', 'user.email'])
  .getMany();
```

### Proper Indexing

```typescript
@Entity()
@Index(['email'])  // Single column index
@Index(['status', 'createdAt'])  // Composite index for common query
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  @Index()  // Index on frequently searched field
  email: string;

  @Column()
  status: string;

  @CreateDateColumn()
  createdAt: Date;
}
```

### Query Logging for Debugging

```typescript
TypeOrmModule.forRoot({
  // ...
  logging: ['query', 'error'],
  logger: 'advanced-console',
});
```

## Common Pitfalls

1. **Eager loading everything**: Only load relations you need.
2. **Missing indexes on WHERE columns**: Index columns used in filters.
3. **SELECT ***: Always select only needed columns.

## Best Practices

- Profile queries before optimizing
- Use relations for related data loading
- Index columns used in WHERE, ORDER BY, JOIN
- Use query builder for complex queries
- Log queries in development

## Summary

Query optimization eliminates N+1 problems through eager loading, improves lookup speed with indexes, and reduces data transfer by selecting specific fields. Use query builder for complex queries and log queries during development.

## Resources

- [Database](https://docs.nestjs.com/techniques/database) — Database optimization techniques

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*