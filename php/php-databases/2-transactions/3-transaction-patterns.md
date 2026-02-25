---
source_course: "php-databases"
source_lesson: "php-databases-transaction-patterns"
---

# Advanced Transaction Patterns

## Introduction

Beyond basic begin/commit/rollback, real applications need patterns for nested transactions, idempotent operations, and transactional outbox. These patterns solve the hard problems of data consistency in distributed and concurrent systems.

## Key Concepts

- **Savepoint**: A named marker within a transaction that allows partial rollback without aborting the entire transaction.
- **Idempotency Key**: A unique identifier that ensures an operation produces the same result when executed multiple times.
- **Transactional Outbox**: A pattern that stores events in the database within the same transaction as the data change, ensuring consistency between state and events.

## Real World Context

Payment processing is a classic case: if a user double-clicks a "Pay" button, the system must not charge them twice. An idempotency key combined with a transaction ensures exactly-once semantics even under network retries.

## Deep Dive

Savepoints provide nested transaction behavior that PDO does not support natively:

```php
<?php
class NestedTransactionManager {
    private int $depth = 0;
    
    public function __construct(private PDO $pdo) {}
    
    public function begin(): void {
        if ($this->depth === 0) {
            $this->pdo->beginTransaction();
        } else {
            $this->pdo->exec("SAVEPOINT sp_{$this->depth}");
        }
        $this->depth++;
    }
    
    public function commit(): void {
        $this->depth--;
        if ($this->depth === 0) {
            $this->pdo->commit();
        } else {
            $this->pdo->exec("RELEASE SAVEPOINT sp_{$this->depth}");
        }
    }
    
    public function rollback(): void {
        $this->depth--;
        if ($this->depth === 0) {
            $this->pdo->rollBack();
        } else {
            $this->pdo->exec("ROLLBACK TO SAVEPOINT sp_{$this->depth}");
        }
    }
}
```

This allows service methods to declare transactions without worrying about whether the caller already started one.

Idempotent operations prevent duplicate processing:

```php
<?php
function processPayment(PDO $pdo, string $idempotencyKey, int $userId, float $amount): array {
    $pdo->beginTransaction();
    try {
        // Check if already processed
        $stmt = $pdo->prepare(
            'SELECT id, status FROM payments WHERE idempotency_key = :key FOR UPDATE'
        );
        $stmt->execute(['key' => $idempotencyKey]);
        $existing = $stmt->fetch();
        
        if ($existing) {
            $pdo->commit();
            return $existing; // Already processed, return same result
        }
        
        // Process the payment
        $stmt = $pdo->prepare(
            'INSERT INTO payments (idempotency_key, user_id, amount, status) VALUES (:key, :uid, :amount, :status)'
        );
        $stmt->execute([
            'key' => $idempotencyKey,
            'uid' => $userId,
            'amount' => $amount,
            'status' => 'completed',
        ]);
        
        $pdo->prepare('UPDATE accounts SET balance = balance - :amount WHERE user_id = :uid')
            ->execute(['amount' => $amount, 'uid' => $userId]);
        
        $pdo->commit();
        return ['id' => $pdo->lastInsertId(), 'status' => 'completed'];
    } catch (Throwable $e) {
        $pdo->rollBack();
        throw $e;
    }
}
```

The transactional outbox pattern ensures events are published reliably:

```php
<?php
function createOrder(PDO $pdo, array $orderData): int {
    $pdo->beginTransaction();
    try {
        $stmt = $pdo->prepare(
            'INSERT INTO orders (user_id, total) VALUES (:uid, :total)'
        );
        $stmt->execute(['uid' => $orderData['user_id'], 'total' => $orderData['total']]);
        $orderId = (int) $pdo->lastInsertId();
        
        // Store event in same transaction
        $pdo->prepare(
            'INSERT INTO outbox_events (aggregate_type, aggregate_id, event_type, payload) VALUES (?, ?, ?, ?)'
        )->execute([
            'order',
            $orderId,
            'order.created',
            json_encode(['order_id' => $orderId, 'total' => $orderData['total']]),
        ]);
        
        $pdo->commit();
        return $orderId;
    } catch (Throwable $e) {
        $pdo->rollBack();
        throw $e;
    }
}
// A separate worker polls outbox_events and publishes to message queue
```

Both the order and its event are committed atomically. A background worker then reads from the outbox and publishes to a message broker.

## Common Pitfalls

1. **Not using FOR UPDATE with idempotency checks** — Without the lock, two concurrent requests with the same key can both pass the existence check and create duplicate records.
2. **Releasing savepoints in the wrong order** — Savepoints form a stack. Releasing or rolling back to a savepoint invalidates all savepoints created after it.

## Best Practices

1. **Use idempotency keys for all payment and financial operations** — This prevents double charges from network retries, user double-clicks, or webhook re-deliveries.
2. **Prefer the transactional outbox over publishing events after commit** — If the event publish fails after commit, the state change is saved but the event is lost. The outbox guarantees both succeed together.

## Summary

- Savepoints enable nested transaction behavior by creating named rollback points.
- Idempotency keys prevent duplicate operations by checking for existing results within a locked transaction.
- The transactional outbox pattern ensures data changes and events are committed atomically.
- Always use `FOR UPDATE` when checking for existing records in idempotent operations.

## Code Examples

**Nested transaction manager using savepoints**

```php
<?php
declare(strict_types=1);

class NestedTransactionManager {
    private int $depth = 0;
    
    public function __construct(private PDO $pdo) {}
    
    public function run(callable $callback): mixed {
        $this->begin();
        try {
            $result = $callback();
            $this->commit();
            return $result;
        } catch (Throwable $e) {
            $this->rollback();
            throw $e;
        }
    }
    
    private function begin(): void {
        if ($this->depth === 0) {
            $this->pdo->beginTransaction();
        } else {
            $this->pdo->exec("SAVEPOINT sp_{$this->depth}");
        }
        $this->depth++;
    }
    
    private function commit(): void {
        $this->depth--;
        if ($this->depth === 0) {
            $this->pdo->commit();
        } else {
            $this->pdo->exec("RELEASE SAVEPOINT sp_{$this->depth}");
        }
    }
    
    private function rollback(): void {
        $this->depth--;
        if ($this->depth === 0) {
            $this->pdo->rollBack();
        } else {
            $this->pdo->exec("ROLLBACK TO SAVEPOINT sp_{$this->depth}");
        }
    }
}
?>
```


## Resources

- [MySQL Savepoints](https://dev.mysql.com/doc/refman/8.0/en/savepoint.html) — MySQL savepoint syntax and behavior

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*