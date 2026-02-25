---
source_course: "php-databases"
source_lesson: "php-databases-transaction-basics"
---

# Understanding Transactions

## Introduction

Transactions group multiple database operations into a single atomic unit. Either all operations succeed and are committed, or all fail and are rolled back. This guarantee is the cornerstone of data integrity in any serious application.

## Key Concepts

- **ACID Properties**: Atomicity (all or nothing), Consistency (valid state transitions), Isolation (concurrent transactions don't interfere), Durability (committed data persists through crashes).
- **Commit**: Permanently applies all operations in the transaction.
- **Rollback**: Reverts all operations in the transaction, restoring the previous state.

## Real World Context

Consider a bank transfer: deducting from one account and crediting another must happen together. If the server crashes between the two operations, a transaction ensures neither change persists, preventing money from vanishing.

## Deep Dive

A basic PDO transaction wraps operations in `beginTransaction()` / `commit()` with a `rollBack()` in the catch block:

```php
<?php
try {
    $pdo->beginTransaction();
    
    $stmt = $pdo->prepare(
        'UPDATE accounts SET balance = balance - :amount WHERE id = :from'
    );
    $stmt->execute(['amount' => 100, 'from' => 1]);
    
    $stmt = $pdo->prepare(
        'UPDATE accounts SET balance = balance + :amount WHERE id = :to'
    );
    $stmt->execute(['amount' => 100, 'to' => 2]);
    
    $pdo->commit();
} catch (Exception $e) {
    $pdo->rollBack();
    throw $e;
}
```

Without a transaction, a crash between the two statements leaves the data inconsistent:

```php
<?php
// DANGEROUS: No transaction
$pdo->exec('UPDATE accounts SET balance = balance - 100 WHERE id = 1');
// Server crash here = money disappears!
$pdo->exec('UPDATE accounts SET balance = balance + 100 WHERE id = 2');
```

You can check whether a transaction is active and use savepoints for partial rollbacks:

```php
<?php
if ($pdo->inTransaction()) {
    // Already in a transaction
}

// Savepoints for partial rollback
$pdo->beginTransaction();
// ... operations ...
$pdo->exec('SAVEPOINT sp1');
// ... more operations ...
$pdo->exec('ROLLBACK TO sp1'); // Undo only since savepoint
$pdo->commit(); // Commit everything before savepoint
```

By default, PDO auto-commits each statement. `beginTransaction()` disables auto-commit until `commit()` or `rollBack()` is called:

```php
<?php
// Auto-commit: each statement commits immediately
$pdo->exec('INSERT INTO logs VALUES (...)');

// Transaction: nothing commits until you say so
$pdo->beginTransaction();
$pdo->exec('INSERT INTO logs VALUES (...)');
$pdo->commit(); // Now committed
```

## Common Pitfalls

1. **Forgetting to rollback on exception** — If an exception occurs and you do not call `rollBack()`, the transaction stays open and can block other connections until the PHP process ends.
2. **Nesting beginTransaction() calls** — PDO does not support nested transactions natively. Calling `beginTransaction()` twice throws an exception. Use savepoints for nested behavior.

## Best Practices

1. **Always use try/catch/rollback** — Every transaction should be wrapped in a try block with rollback in the catch to guarantee cleanup.
2. **Keep transactions short** — Long-running transactions hold locks and block other queries. Do computation before the transaction, then execute the database operations quickly.

## Summary

- Transactions guarantee atomicity: all operations succeed together or fail together.
- Always wrap transactions in try/catch with rollback on failure.
- Use savepoints for partial rollback within a transaction.
- Keep transactions as short as possible to minimize lock contention.

## Code Examples

**Transaction manager with automatic rollback**

```php
<?php
declare(strict_types=1);

class TransactionManager {
    public function __construct(private PDO $pdo) {}
    
    /**
     * @template T
     * @param callable(): T $callback
     * @return T
     */
    public function transaction(callable $callback): mixed {
        $this->pdo->beginTransaction();
        try {
            $result = $callback();
            $this->pdo->commit();
            return $result;
        } catch (Throwable $e) {
            $this->pdo->rollBack();
            throw $e;
        }
    }
}

// Usage
$tx = new TransactionManager($pdo);
$orderId = $tx->transaction(function() use ($pdo, $userId, $items) {
    $pdo->prepare('INSERT INTO orders (user_id) VALUES (?)')
        ->execute([$userId]);
    $orderId = $pdo->lastInsertId();
    foreach ($items as $item) {
        $pdo->prepare('INSERT INTO order_items (order_id, product_id, qty) VALUES (?, ?, ?)')
            ->execute([$orderId, $item['product_id'], $item['qty']]);
    }
    return $orderId;
});
?>
```


## Resources

- [PDO Transactions](https://www.php.net/manual/en/pdo.transactions.php) — PDO transactions documentation

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*