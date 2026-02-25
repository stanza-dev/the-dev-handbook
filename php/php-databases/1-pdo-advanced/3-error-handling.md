---
source_course: "php-databases"
source_lesson: "php-databases-pdo-error-handling"
---

# PDO Error Handling & Debugging

## Introduction

Robust error handling transforms cryptic database failures into actionable diagnostics. PHP 8.5 makes exception-based error handling the undisputed standard, but understanding how to catch, log, and debug PDO errors separates production-ready code from tutorial-level code.

## Key Concepts

- **PDOException**: The exception class thrown by PDO on errors (default since PHP 8.0).
- **SQLSTATE Code**: A five-character standard error code from the database driver, accessible via `$exception->getCode()`.
- **Error Info Array**: A three-element array from `$stmt->errorInfo()` containing SQLSTATE, driver code, and driver message.

## Real World Context

In production, database errors happen constantly: duplicate key violations, connection timeouts, deadlocks, constraint failures. Your application needs to distinguish between retryable errors (deadlocks), user-caused errors (duplicate email), and system errors (connection lost) to respond appropriately.

## Deep Dive

Since PHP 8.0, PDO throws `PDOException` on all errors by default. Every database error becomes a catchable exception:

```php
<?php
try {
    $stmt = $pdo->prepare('INSERT INTO users (email) VALUES (:email)');
    $stmt->execute(['email' => 'john@example.com']);
} catch (PDOException $e) {
    $sqlState = $e->getCode(); // e.g., '23000'
    
    match ($sqlState) {
        '23000' => throw new DuplicateEntryException('Email already exists'),
        '23503' => throw new ForeignKeyException('Referenced record not found'),
        '40001' => throw new DeadlockException('Deadlock detected, retry'),
        'HY000' => throw new DatabaseException('General database error'),
        default => throw $e,
    };
}
```

Common SQLSTATE codes you will encounter:

```php
<?php
// Constraint violations
// 23000 - Integrity constraint (duplicate key, NOT NULL)
// 23503 - Foreign key violation

// Concurrency
// 40001 - Serialization failure / deadlock
// 40P01 - PostgreSQL deadlock detected

// Connection errors
// 08006 - Connection failure
// 08001 - Unable to connect

// Syntax/access
// 42S02 - Table not found
// 42000 - Syntax error or access violation
```

A production error handler wraps these patterns:

```php
<?php
class DatabaseErrorHandler {
    private const RETRYABLE = ['40001', '40P01'];
    
    public function __construct(private LoggerInterface $logger) {}
    
    public function executeWithRetry(
        callable $operation,
        int $maxRetries = 3
    ): mixed {
        $attempt = 0;
        while (true) {
            try {
                return $operation();
            } catch (PDOException $e) {
                $attempt++;
                if (
                    $attempt >= $maxRetries ||
                    !in_array($e->getCode(), self::RETRYABLE, true)
                ) {
                    $this->logger->error('Database error', [
                        'sqlstate' => $e->getCode(),
                        'attempt' => $attempt,
                    ]);
                    throw $e;
                }
                usleep((int) (100_000 * (2 ** ($attempt - 1))));
            }
        }
    }
}
```

For debugging during development, a query logger is invaluable:

```php
<?php
class DebuggablePDO extends PDO {
    private array $queryLog = [];
    
    public function prepare(string $query, array $options = []): PDOStatement|false {
        $this->queryLog[] = [
            'query' => $query,
            'time' => microtime(true),
        ];
        return parent::prepare($query, $options);
    }
    
    public function getQueryLog(): array {
        return $this->queryLog;
    }
}
```

Never expose raw database error messages to end users. They reveal table names, column names, and structure.

## Common Pitfalls

1. **Catching PDOException and ignoring it** — Silent failures lead to data corruption. Always log and either re-throw or handle explicitly.
2. **Exposing error messages to users** — Raw PDO errors contain query fragments that aid attackers.

## Best Practices

1. **Map SQLSTATE codes to domain exceptions** — Translate database errors into application-level exceptions that controllers can handle cleanly.
2. **Implement retry logic for deadlocks** — Deadlocks are normal in concurrent systems; a retry loop with exponential backoff resolves most cases.

## Summary

- Since PHP 8.0, PDO throws exceptions by default; no manual ERRMODE configuration needed.
- SQLSTATE codes let you distinguish between duplicate keys, foreign key violations, and deadlocks.
- Always log database errors with context and never expose raw messages to end users.
- Implement retry logic for transient errors like deadlocks.

## Code Examples

**Error handler with retry logic for deadlocks**

```php
<?php
declare(strict_types=1);

class DatabaseErrorHandler {
    private const RETRYABLE_STATES = ['40001', '40P01'];
    
    public function __construct(private LoggerInterface $logger) {}
    
    public function executeWithRetry(
        callable $operation,
        int $maxRetries = 3
    ): mixed {
        $attempt = 0;
        while (true) {
            try {
                return $operation();
            } catch (PDOException $e) {
                $attempt++;
                if (
                    $attempt >= $maxRetries ||
                    !in_array($e->getCode(), self::RETRYABLE_STATES, true)
                ) {
                    $this->logger->error('DB error', [
                        'sqlstate' => $e->getCode(),
                        'attempt' => $attempt,
                    ]);
                    throw $e;
                }
                usleep((int) (100_000 * (2 ** ($attempt - 1))));
            }
        }
    }
}
?>
```


## Resources

- [PDO Error Handling](https://www.php.net/manual/en/pdo.error-handling.php) — PDO error handling documentation

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*