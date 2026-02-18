---
source_course: "php-databases"
source_lesson: "php-databases-isolation-locking"
---

# Isolation Levels & Locking

Isolation levels control how transactions interact with each other.

## Isolation Levels

```php
<?php
// Set isolation level (MySQL)
$pdo->exec('SET TRANSACTION ISOLATION LEVEL READ COMMITTED');
$pdo->beginTransaction();

// Levels from least to most restrictive:
// READ UNCOMMITTED - See uncommitted changes (dirty reads)
// READ COMMITTED - Only see committed data
// REPEATABLE READ - Same query returns same results (MySQL default)
// SERIALIZABLE - Full isolation (slowest)
```

## Concurrency Problems

### Dirty Read
```php
<?php
// Transaction A (READ UNCOMMITTED)
// Sees Transaction B's uncommitted changes
// If B rolls back, A read invalid data!
```

### Non-Repeatable Read
```php
<?php
// Transaction A reads row, gets $100
// Transaction B updates row to $50, commits
// Transaction A reads again, gets $50
// Same query, different result!
```

### Phantom Read
```php
<?php
// Transaction A counts rows WHERE status = 'active', gets 10
// Transaction B inserts new active row, commits
// Transaction A counts again, gets 11
// New rows appeared!
```

## Row Locking

```php
<?php
$pdo->beginTransaction();

// SELECT ... FOR UPDATE locks the rows
$stmt = $pdo->prepare(
    'SELECT balance FROM accounts WHERE id = :id FOR UPDATE'
);
$stmt->execute(['id' => 1]);
$balance = $stmt->fetchColumn();

// Other transactions wait here until we commit/rollback

if ($balance >= $amount) {
    $stmt = $pdo->prepare(
        'UPDATE accounts SET balance = balance - :amount WHERE id = :id'
    );
    $stmt->execute(['amount' => $amount, 'id' => 1]);
}

$pdo->commit();  // Lock released
```

## Optimistic Locking

```php
<?php
class OptimisticLockException extends Exception {}

// Add version column to table
// version INT DEFAULT 1

function updateWithOptimisticLock(PDO $pdo, int $id, array $data): void {
    // Read current version
    $stmt = $pdo->prepare('SELECT version FROM products WHERE id = :id');
    $stmt->execute(['id' => $id]);
    $version = $stmt->fetchColumn();
    
    // Update only if version matches
    $stmt = $pdo->prepare(
        'UPDATE products 
         SET name = :name, price = :price, version = version + 1
         WHERE id = :id AND version = :version'
    );
    $stmt->execute([
        'id' => $id,
        'name' => $data['name'],
        'price' => $data['price'],
        'version' => $version,
    ]);
    
    if ($stmt->rowCount() === 0) {
        throw new OptimisticLockException('Record was modified by another process');
    }
}
```

## Code Examples

**Transaction manager with locking support**

```php
<?php
declare(strict_types=1);

// Transaction manager with automatic rollback
class TransactionManager {
    public function __construct(
        private PDO $pdo
    ) {}
    
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
    
    public function withLock(string $table, int $id, callable $callback): mixed {
        return $this->transaction(function() use ($table, $id, $callback) {
            // Acquire row lock
            $stmt = $this->pdo->prepare(
                "SELECT * FROM $table WHERE id = :id FOR UPDATE"
            );
            $stmt->execute(['id' => $id]);
            $row = $stmt->fetch();
            
            if (!$row) {
                throw new RuntimeException('Record not found');
            }
            
            return $callback($row);
        });
    }
}

// Usage
$txManager = new TransactionManager($pdo);

// Simple transaction
$orderId = $txManager->transaction(function() use ($pdo, $items) {
    $pdo->prepare('INSERT INTO orders (user_id) VALUES (:user_id)')
        ->execute(['user_id' => $userId]);
    $orderId = $pdo->lastInsertId();
    
    foreach ($items as $item) {
        $pdo->prepare('INSERT INTO order_items (order_id, product_id, quantity) VALUES (?, ?, ?)')
            ->execute([$orderId, $item['product_id'], $item['quantity']]);
    }
    
    return $orderId;
});

// With row locking
$txManager->withLock('accounts', 1, function($account) use ($pdo, $amount) {
    if ($account['balance'] < $amount) {
        throw new InsufficientFundsException();
    }
    
    $pdo->prepare('UPDATE accounts SET balance = balance - ? WHERE id = ?')
        ->execute([$amount, $account['id']]);
});
?>
```


---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*