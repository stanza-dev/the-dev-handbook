---
source_course: "php-databases"
source_lesson: "php-databases-transaction-basics"
---

# Understanding Transactions

Transactions group multiple operations into a single atomic unit. Either all succeed, or all fail.

## ACID Properties

- **Atomicity**: All or nothing
- **Consistency**: Database remains valid
- **Isolation**: Concurrent transactions don't interfere
- **Durability**: Committed data persists

## Basic Transaction

```php
<?php
try {
    $pdo->beginTransaction();
    
    // Transfer money between accounts
    $stmt = $pdo->prepare('UPDATE accounts SET balance = balance - :amount WHERE id = :from');
    $stmt->execute(['amount' => 100, 'from' => 1]);
    
    $stmt = $pdo->prepare('UPDATE accounts SET balance = balance + :amount WHERE id = :to');
    $stmt->execute(['amount' => 100, 'to' => 2]);
    
    $pdo->commit();  // Both changes applied
    
} catch (Exception $e) {
    $pdo->rollBack();  // Both changes reverted
    throw $e;
}
```

## Why Transactions Matter

```php
<?php
// Without transaction - Inconsistent data possible!
$pdo->exec('UPDATE accounts SET balance = balance - 100 WHERE id = 1');
// Server crash here = money disappears!
$pdo->exec('UPDATE accounts SET balance = balance + 100 WHERE id = 2');

// With transaction - Atomic operation
$pdo->beginTransaction();
$pdo->exec('UPDATE accounts SET balance = balance - 100 WHERE id = 1');
// Server crash here = transaction rolled back automatically
$pdo->exec('UPDATE accounts SET balance = balance + 100 WHERE id = 2');
$pdo->commit();
```

## Transaction State

```php
<?php
// Check if in transaction
if ($pdo->inTransaction()) {
    // Handle differently
}

// Nested transactions (savepoints)
$pdo->beginTransaction();
// ... operations ...

$pdo->exec('SAVEPOINT sp1');
// ... more operations ...

// Partial rollback
$pdo->exec('ROLLBACK TO sp1');

// Continue and commit
$pdo->commit();
```

## Auto-Commit Behavior

```php
<?php
// By default, each query auto-commits
$pdo->exec('INSERT INTO logs VALUES (...)');  // Committed immediately

// beginTransaction() disables auto-commit
$pdo->beginTransaction();
$pdo->exec('INSERT INTO logs VALUES (...)');  // Not committed yet
$pdo->commit();  // Now committed

// Or
$pdo->rollBack();  // Discarded
```

## Resources

- [PDO Transactions](https://www.php.net/manual/en/pdo.transactions.php) — PDO transactions documentation

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*