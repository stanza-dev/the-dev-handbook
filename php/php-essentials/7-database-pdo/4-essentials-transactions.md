---
source_course: "php-essentials"
source_lesson: "php-essentials-transactions"
---

# Database Transactions

## Introduction

When a database operation involves multiple queries that must all succeed or all fail together, you need transactions. Without them, a failure halfway through could leave your data in an inconsistent state — money deducted from one account but never credited to another, an order created but inventory never updated. This lesson covers the ACID properties, PDO transaction methods, and practical patterns for safe multi-step operations.

## Key Concepts

- **Transaction**: A group of database operations treated as a single atomic unit.
- **ACID properties**: Atomicity (all or nothing), Consistency (data remains valid), Isolation (operations don't interfere), Durability (committed changes persist).
- **`beginTransaction()`**: Starts a new transaction. Subsequent queries are not committed until explicitly requested.
- **`commit()`**: Saves all changes made during the transaction.
- **`rollBack()`**: Undoes all changes made during the transaction.
- **`inTransaction()`**: Returns `true` if a transaction is currently active.

## Real World Context

Transactions are essential in any application that handles money, inventory, or coordinated multi-table updates. Think of a bank transfer: deducting from Account A and crediting Account B must happen together — if the credit fails after the deduction, the money disappears. E-commerce order processing, user registration with profile creation, and batch data imports all require transactions to maintain data integrity.

## Deep Dive

### What is a Transaction?

A transaction groups operations so they are treated as a single unit. The ACID properties guarantee:

- **Atomicity**: All operations succeed, or none of them do.
- **Consistency**: The database moves from one valid state to another.
- **Isolation**: Concurrent transactions do not interfere with each other.
- **Durability**: Once committed, changes survive even a server crash.

### Basic Transaction

The standard pattern is try-commit-catch-rollback:

```php
<?php
try {
    $pdo->beginTransaction();
    
    // Multiple operations
    $pdo->exec("UPDATE accounts SET balance = balance - 100 WHERE id = 1");
    $pdo->exec("UPDATE accounts SET balance = balance + 100 WHERE id = 2");
    
    // If we get here, commit all changes
    $pdo->commit();
    
} catch (Exception $e) {
    // Something went wrong, undo everything
    $pdo->rollBack();
    throw $e;
}
```

If any query fails and throws an exception, `rollBack()` undoes all changes made since `beginTransaction()`. Re-throwing the exception ensures the error is not silently swallowed.

### Practical Example: Order Processing

This real-world example creates an order, adds line items, and updates inventory atomically:

```php
<?php
function createOrder(PDO $pdo, int $userId, array $items): int {
    $pdo->beginTransaction();
    
    try {
        // 1. Create the order
        $stmt = $pdo->prepare(
            'INSERT INTO orders (user_id, status, created_at) 
             VALUES (:user_id, :status, NOW())'
        );
        $stmt->execute([':user_id' => $userId, ':status' => 'pending']);
        $orderId = (int) $pdo->lastInsertId();
        
        // 2. Add order items and update inventory
        $itemStmt = $pdo->prepare(
            'INSERT INTO order_items (order_id, product_id, quantity, price) 
             VALUES (:order_id, :product_id, :quantity, :price)'
        );
        
        $stockStmt = $pdo->prepare(
            'UPDATE products SET stock = stock - :qty WHERE id = :id AND stock >= :qty'
        );
        
        foreach ($items as $item) {
            // Reduce stock
            $stockStmt->execute([
                ':qty' => $item['quantity'],
                ':id' => $item['product_id']
            ]);
            
            // Check if stock was actually reduced
            if ($stockStmt->rowCount() === 0) {
                throw new Exception("Insufficient stock for product {$item['product_id']}");
            }
            
            // Add order item
            $itemStmt->execute([
                ':order_id' => $orderId,
                ':product_id' => $item['product_id'],
                ':quantity' => $item['quantity'],
                ':price' => $item['price']
            ]);
        }
        
        $pdo->commit();
        return $orderId;
        
    } catch (Exception $e) {
        $pdo->rollBack();
        throw $e;
    }
}
```

Notice how `rowCount() === 0` detects insufficient stock. If any product is out of stock, the exception triggers a rollback, undoing the order creation and any prior stock changes.

### Transaction Methods

PDO provides four transaction-related methods:

```php
<?php
$pdo->beginTransaction();  // Start transaction
$pdo->commit();            // Save all changes
$pdo->rollBack();          // Undo all changes
$pdo->inTransaction();     // Check if in transaction (bool)
```

Use `inTransaction()` to avoid calling `beginTransaction()` when one is already active, which would throw an exception.

### Nested Transaction Pattern

PDO does not support true nested transactions, but you can simulate them with a depth counter:

```php
<?php
class TransactionManager {
    private int $level = 0;
    
    public function __construct(private PDO $pdo) {}
    
    public function begin(): void {
        if ($this->level === 0) {
            $this->pdo->beginTransaction();
        }
        $this->level++;
    }
    
    public function commit(): void {
        $this->level--;
        if ($this->level === 0) {
            $this->pdo->commit();
        }
    }
    
    public function rollback(): void {
        $this->level = 0;
        $this->pdo->rollBack();
    }
}
```

This pattern is useful in larger applications where service methods may each want transactional behavior but can also be composed together.

## Common Pitfalls

1. **Forgetting to rollback on exception** — Always wrap transactions in try-catch with `rollBack()` in the catch block. An uncommitted transaction left open can lock database rows.
2. **Calling `beginTransaction()` when one is already active** — PDO throws an exception if you start a second transaction. Check with `inTransaction()` first, or use the nested transaction pattern.
3. **Performing non-transactional operations inside a transaction** — Sending emails or calling external APIs inside a transaction block means those side effects cannot be rolled back. Move them after the `commit()`.

## Best Practices

1. **Keep transactions short** — Hold transactions open for as little time as possible to minimize lock contention. Do all preparation before `beginTransaction()` and all side effects after `commit()`.
2. **Always re-throw after rollback** — Catching an exception, rolling back, and then silently continuing hides errors. Always re-throw or log the exception.
3. **Use transactions for any multi-query operation** — If two or more queries must succeed together, wrap them in a transaction, even if failure seems unlikely.

## Summary

- Transactions group multiple queries into an atomic unit: all succeed or all fail.
- Use `beginTransaction()`, `commit()`, and `rollBack()` with a try-catch pattern.
- The ACID properties (Atomicity, Consistency, Isolation, Durability) guarantee data integrity.
- Always rollback on exception and re-throw the error.
- Keep transactions short and move side effects (emails, API calls) outside the transaction block.

## Code Examples

**Bank transfer function demonstrating transactions with balance checking, FOR UPDATE locking, and transfer logging — all rolled back if any step fails**

```php
<?php
// Bank transfer with transaction safety
function transferFunds(
    PDO $pdo,
    int $fromAccountId,
    int $toAccountId,
    float $amount
): void {
    if ($amount <= 0) {
        throw new InvalidArgumentException('Transfer amount must be positive');
    }
    
    $pdo->beginTransaction();
    
    try {
        // 1. Check sender has sufficient balance
        $stmt = $pdo->prepare(
            'SELECT balance FROM accounts WHERE id = :id FOR UPDATE'
        );
        $stmt->execute([':id' => $fromAccountId]);
        $sender = $stmt->fetch();
        
        if (!$sender || $sender['balance'] < $amount) {
            throw new RuntimeException('Insufficient funds');
        }
        
        // 2. Deduct from sender
        $pdo->prepare(
            'UPDATE accounts SET balance = balance - :amount WHERE id = :id'
        )->execute([':amount' => $amount, ':id' => $fromAccountId]);
        
        // 3. Credit to receiver
        $pdo->prepare(
            'UPDATE accounts SET balance = balance + :amount WHERE id = :id'
        )->execute([':amount' => $amount, ':id' => $toAccountId]);
        
        // 4. Log the transfer
        $pdo->prepare(
            'INSERT INTO transfers (from_id, to_id, amount, created_at)
             VALUES (:from, :to, :amount, NOW())'
        )->execute([
            ':from' => $fromAccountId,
            ':to' => $toAccountId,
            ':amount' => $amount
        ]);
        
        $pdo->commit();
        
    } catch (Exception $e) {
        $pdo->rollBack();
        throw $e;
    }
}
?>
```


## Resources

- [PDO Transactions](https://www.php.net/manual/en/pdo.transactions.php) — Official guide to PDO transactions

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*