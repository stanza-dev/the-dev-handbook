---
source_course: "nextjs-api-patterns"
source_lesson: "nextjs-api-patterns-njs-transactions"
---

# Database Transactions

## Introduction

Some operations need to succeed or fail together. Creating an order should create order items, update inventory, and charge the customer—if any step fails, everything should roll back.

## Key Concepts

**Transaction** ensures ACID properties:
- **Atomicity**: All operations succeed or all fail
- **Consistency**: Database stays in a valid state
- **Isolation**: Concurrent transactions don't interfere
- **Durability**: Committed changes persist

## Real World Context

Transactions prevent:
- Money transferred but not received
- Orders created without inventory update
- Partial data corruption
- Race conditions in concurrent requests

## Deep Dive

### Interactive Transactions

```typescript
export async function POST(request: Request) {
  const { userId, items } = await request.json();

  const order = await prisma.$transaction(async (tx) => {
    // 1. Create the order
    const order = await tx.order.create({
      data: {
        userId,
        status: 'pending',
      },
    });

    // 2. Create order items and update inventory
    for (const item of items) {
      await tx.orderItem.create({
        data: {
          orderId: order.id,
          productId: item.productId,
          quantity: item.quantity,
        },
      });

      // Update inventory (throws if not enough stock)
      const product = await tx.product.update({
        where: { id: item.productId },
        data: {
          stock: { decrement: item.quantity },
        },
      });

      if (product.stock < 0) {
        throw new Error(`Insufficient stock for ${item.productId}`);
      }
    }

    // 3. Calculate and update total
    const total = await calculateOrderTotal(tx, order.id);
    return tx.order.update({
      where: { id: order.id },
      data: { total },
    });
  });

  return Response.json(order, { status: 201 });
}
```

### Batch Transactions

For multiple independent operations:

```typescript
const [user, posts, settings] = await prisma.$transaction([
  prisma.user.create({ data: userData }),
  prisma.post.createMany({ data: postsData }),
  prisma.settings.create({ data: settingsData }),
]);
```

### Error Handling

```typescript
export async function POST(request: Request) {
  try {
    const result = await prisma.$transaction(async (tx) => {
      // Operations...
    });
    return Response.json(result, { status: 201 });
  } catch (error) {
    if (error instanceof Error) {
      if (error.message.includes('Insufficient stock')) {
        return Response.json(
          { error: 'Some items are out of stock' },
          { status: 400 }
        );
      }
    }
    console.error('Transaction failed:', error);
    return Response.json(
      { error: 'Failed to process order' },
      { status: 500 }
    );
  }
}
```

## Common Pitfalls

1. **Long-running transactions**: They hold database locks. Keep transactions short.

2. **External API calls inside transactions**: If an external call fails, the DB rolls back but the external action doesn't.

3. **Not using `tx` inside transaction**: Always use the transaction client (`tx`), not the global `prisma`.

## Best Practices

- **Keep transactions short**: Minimize lock time
- **Do external calls outside transactions**: Call APIs before or after, not during
- **Use appropriate isolation levels**: Higher isolation = more safety but less concurrency
- **Handle all error cases**: Return meaningful errors, not just 500

## Summary

Transactions ensure multiple database operations succeed or fail together. Use `prisma.$transaction()` with an async callback for complex logic, or pass an array for batch operations. Always use the `tx` client inside transactions and keep them short.

## Resources

- [Prisma Transactions](https://www.prisma.io/docs/concepts/components/prisma-client/transactions) — Prisma transaction documentation

---

> 📘 *This lesson is part of the [Next.js API Routes & Patterns](https://stanza.dev/courses/nextjs-api-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*