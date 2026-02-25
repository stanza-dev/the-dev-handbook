---
source_course: "php-databases"
source_lesson: "php-databases-isolation-locking"
---

# Isolation Levels & Locking

## Introduction

Isolation levels control how much one transaction can see of another's uncommitted work. Combined with locking strategies, they determine the trade-off between data correctness and concurrent throughput.

## Key Concepts

- **Isolation Level**: A database setting that defines visibility rules between concurrent transactions.
- **Pessimistic Locking**: Explicitly locking rows with `SELECT ... FOR UPDATE` to prevent concurrent modification.
- **Optimistic Locking**: Using a version column to detect conflicts at update time without holding locks.

## Real World Context

An e-commerce site processing thousands of concurrent orders needs isolation levels tuned correctly. Too strict and throughput drops; too loose and customers see phantom inventory or double-spend their account balance.

## Deep Dive

The four SQL standard isolation levels, from least to most restrictive:

```php
<?php
// Set isolation level before starting a transaction (MySQL)
$pdo->exec('SET TRANSACTION ISOLATION LEVEL READ COMMITTED');
$pdo->beginTransaction();

// READ UNCOMMITTED - See uncommitted changes (dirty reads)
// READ COMMITTED   - Only see committed data
// REPEATABLE READ  - Same query returns same results in a transaction
// SERIALIZABLE     - Full isolation, transactions behave as if sequential
```

Important: default isolation levels differ between databases. MySQL/InnoDB defaults to `REPEATABLE READ`, while PostgreSQL defaults to `READ COMMITTED`. This means identical code may behave differently across databases.

Note: in PHP 8.5, the `PDO::PGSQL_TRANSACTION_*` constants are deprecated. Use the SQL-level `SET TRANSACTION ISOLATION LEVEL` statement instead.

Each level prevents progressively more concurrency anomalies:

```php
<?php
// Dirty Read: seeing uncommitted changes
// Transaction A sees B's uncommitted INSERT
// If B rolls back, A read data that never existed

// Non-Repeatable Read: same row, different values
// A reads row -> B updates and commits -> A reads again, different value

// Phantom Read: new rows appear
// A counts WHERE status='active' -> B inserts new active row -> A counts again, more rows
```

Pessimistic locking with `SELECT ... FOR UPDATE` locks rows until the transaction ends:

```php
<?php
$pdo->beginTransaction();

$stmt = $pdo->prepare(
    'SELECT balance FROM accounts WHERE id = :id FOR UPDATE'
);
$stmt->execute(['id' => 1]);
$balance = $stmt->fetchColumn();

// Other transactions trying to read this row FOR UPDATE will wait

if ($balance >= $amount) {
    $stmt = $pdo->prepare(
        'UPDATE accounts SET balance = balance - :amount WHERE id = :id'
    );
    $stmt->execute(['amount' => $amount, 'id' => 1]);
}

$pdo->commit(); // Lock released
```

Optimistic locking avoids holding database locks by detecting conflicts at write time:

```php
<?php
function updateWithVersion(PDO $pdo, int $id, array $data): void {
    $stmt = $pdo->prepare('SELECT version FROM products WHERE id = :id');
    $stmt->execute(['id' => $id]);
    $version = $stmt->fetchColumn();
    
    $stmt = $pdo->prepare(
        'UPDATE products SET name = :name, price = :price, version = version + 1
         WHERE id = :id AND version = :version'
    );
    $stmt->execute([
        'id' => $id,
        'name' => $data['name'],
        'price' => $data['price'],
        'version' => $version,
    ]);
    
    if ($stmt->rowCount() === 0) {
        throw new OptimisticLockException(
            'Record was modified by another process'
        );
    }
}
```

Optimistic locking is preferred when conflicts are rare. Pessimistic locking is better when conflicts are frequent or the cost of retrying is high.

## Common Pitfalls

1. **Assuming the same default isolation level across databases** — MySQL defaults to REPEATABLE READ; PostgreSQL defaults to READ COMMITTED. Code that works on one may behave differently on the other.
2. **Holding locks during slow operations** — Performing HTTP requests or file I/O inside a transaction with FOR UPDATE blocks other transactions for the entire duration.

## Best Practices

1. **Use READ COMMITTED for most web applications** — It prevents dirty reads without the overhead of REPEATABLE READ, and is the PostgreSQL default for good reason.
2. **Prefer optimistic locking for low-contention scenarios** — A version column avoids holding database locks and scales better under low conflict rates.

## Summary

- Isolation levels control how concurrent transactions see each other's changes.
- MySQL/InnoDB defaults to REPEATABLE READ; PostgreSQL defaults to READ COMMITTED.
- `SELECT ... FOR UPDATE` provides pessimistic locking; a version column provides optimistic locking.
- PHP 8.5 deprecates `PDO::PGSQL_TRANSACTION_*` constants; use SQL statements instead.

## Code Examples

**Transaction manager with pessimistic locking support**

```php
<?php
declare(strict_types=1);

class TransactionManager {
    public function __construct(private PDO $pdo) {}
    
    public function withLock(string $table, int $id, callable $callback): mixed {
        return $this->transaction(function() use ($table, $id, $callback) {
            $stmt = $this->pdo->prepare(
                "SELECT * FROM {$table} WHERE id = :id FOR UPDATE"
            );
            $stmt->execute(['id' => $id]);
            $row = $stmt->fetch();
            if (!$row) {
                throw new RuntimeException('Record not found');
            }
            return $callback($row);
        });
    }
    
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
?>
```


## Resources

- [MySQL Isolation Levels](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html) — MySQL InnoDB isolation levels reference

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*