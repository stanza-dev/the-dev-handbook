---
source_course: "php-essentials"
source_lesson: "php-essentials-transactions"
---

# Database Transactions

Transactions ensure that multiple database operations either all succeed or all fail together, maintaining data integrity.

## What is a Transaction?

A transaction groups operations so they're treated as a single unit:
- **Atomicity**: All or nothing
- **Consistency**: Data remains valid
- **Isolation**: Operations don't interfere
- **Durability**: Changes persist after commit

## Basic Transaction

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

## Practical Example: Order Processing

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

## Transaction Methods

```php
<?php
$pdo->beginTransaction();  // Start transaction
$pdo->commit();            // Save all changes
$pdo->rollBack();          // Undo all changes
$pdo->inTransaction();     // Check if in transaction (bool)
```

## Nested Transaction Pattern

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

## Resources

- [PDO Transactions](https://www.php.net/manual/en/pdo.transactions.php) — Official guide to PDO transactions

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*