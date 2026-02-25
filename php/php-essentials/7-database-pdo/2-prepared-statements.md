---
source_course: "php-essentials"
source_lesson: "php-essentials-prepared-statements"
---

# Prepared Statements

## Introduction

Prepared statements are the only safe way to execute database queries that include user-supplied data. They completely prevent SQL injection by separating SQL logic from data. This lesson covers the prepare-execute-fetch workflow, both named and positional parameters, and all four CRUD operations.

## Key Concepts

- **Prepared statement**: A pre-compiled SQL template with placeholders where user data will be inserted safely.
- **Named parameters (`:name`)**: Placeholders identified by name, making queries more readable.
- **Positional parameters (`?`)**: Placeholders identified by position, useful for simple queries.
- **SQL injection**: An attack where malicious SQL is inserted through user input. Prepared statements prevent this entirely.
- **`lastInsertId()`**: Returns the ID of the most recently inserted row.
- **`rowCount()`**: Returns the number of rows affected by the last statement.

## Real World Context

SQL injection has been the number one web vulnerability for over two decades. Every time your application uses user input in a database query — login forms, search bars, profile updates — you must use prepared statements. There are no exceptions. Frameworks like Laravel and Symfony use PDO prepared statements under the hood for every database query.

## Deep Dive

### How Prepared Statements Work

The process has three distinct steps:

1. **Prepare**: Send the SQL template with placeholders to the database.
2. **Execute**: Send the actual data values separately.
3. **Fetch**: Retrieve the results.

Because the SQL and data are sent separately, the database always knows what is SQL and what is data — they can never be confused.

### Named Parameters (Recommended)

Named parameters use `:name` syntax and make queries self-documenting:

```php
<?php
$stmt = $pdo->prepare(
    'SELECT * FROM users WHERE email = :email AND status = :status'
);

$stmt->execute([
    ':email' => $email,
    ':status' => 'active'
]);

$user = $stmt->fetch();
```

Named parameters are preferred for queries with multiple parameters because the intent of each value is clear.

### Positional Parameters

Positional parameters use `?` and are matched by order:

```php
<?php
$stmt = $pdo->prepare(
    'SELECT * FROM products WHERE price > ? AND category = ?'
);

$stmt->execute([29.99, 'electronics']);
$products = $stmt->fetchAll();
```

Use positional parameters for simple queries with one or two parameters where the meaning is obvious.

### INSERT Operations

Insert new records and retrieve the generated ID:

```php
<?php
$stmt = $pdo->prepare(
    'INSERT INTO users (name, email, password_hash) VALUES (:name, :email, :password)'
);

$stmt->execute([
    ':name' => $name,
    ':email' => $email,
    ':password' => password_hash($password, PASSWORD_DEFAULT)
]);

// Get the ID of the inserted row
$newUserId = $pdo->lastInsertId();
```

`lastInsertId()` is called on the PDO object, not the statement.

### UPDATE Operations

Update records and check how many rows were affected:

```php
<?php
$stmt = $pdo->prepare(
    'UPDATE users SET name = :name, updated_at = NOW() WHERE id = :id'
);

$stmt->execute([
    ':name' => $newName,
    ':id' => $userId
]);

// Check how many rows were affected
$rowsUpdated = $stmt->rowCount();
```

`rowCount()` returns 0 if the WHERE clause matched no rows, which is useful for "not found" handling.

### DELETE Operations

Delete records with the same prepare-execute pattern:

```php
<?php
$stmt = $pdo->prepare('DELETE FROM users WHERE id = :id');
$stmt->execute([':id' => $userId]);

if ($stmt->rowCount() > 0) {
    echo "User deleted successfully";
}
```

### Why Prepared Statements Prevent SQL Injection

Compare unsafe string interpolation with safe prepared statements:

```php
<?php
// DANGEROUS - SQL Injection vulnerable!
$email = "'; DROP TABLE users; --";
$query = "SELECT * FROM users WHERE email = '$email'";
// Results in: SELECT * FROM users WHERE email = ''; DROP TABLE users; --'

// SAFE - Prepared statement
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute([':email' => $email]);
// The malicious input is treated as a literal string, not SQL
```

With prepared statements, the database engine knows the malicious input is data, not SQL commands. The attack is neutralized automatically.

## Common Pitfalls

1. **Concatenating user input into SQL strings** — Never build queries with string concatenation or interpolation. Even for "just this one query," always use prepared statements.
2. **Reusing `lastInsertId()` after multiple inserts** — `lastInsertId()` only returns the ID from the most recent INSERT. If you need IDs from a batch, call it after each individual insert.
3. **Mixing named and positional parameters** — A single query must use either all named (`:param`) or all positional (`?`) parameters. Mixing them causes an error.

## Best Practices

1. **Use named parameters for readability** — `:email` is more descriptive than `?` when a query has multiple parameters.
2. **Use `password_hash()` for passwords** — Never store plain-text passwords. Use `password_hash($pw, PASSWORD_DEFAULT)` for hashing and `password_verify()` for checking.
3. **Build a repository pattern** — Encapsulate database queries in dedicated repository classes to keep SQL out of your controllers and business logic.

## Summary

- Prepared statements separate SQL structure from data, completely preventing SQL injection.
- Named parameters (`:name`) are more readable; positional parameters (`?`) are simpler for short queries.
- Use `lastInsertId()` after INSERT to get the new row's ID.
- Use `rowCount()` after UPDATE/DELETE to check how many rows were affected.
- Never concatenate user input into SQL strings, no matter how safe it seems.

## Code Examples

**Repository pattern encapsulating all CRUD operations with prepared statements and named parameters**

```php
<?php
// Repository pattern with prepared statements for all CRUD operations
class UserRepository {
    public function __construct(private PDO $pdo) {}
    
    public function findById(int $id): ?array {
        $stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
        $stmt->execute([':id' => $id]);
        $user = $stmt->fetch();
        return $user ?: null;
    }
    
    public function findByEmail(string $email): ?array {
        $stmt = $this->pdo->prepare('SELECT * FROM users WHERE email = :email');
        $stmt->execute([':email' => $email]);
        $user = $stmt->fetch();
        return $user ?: null;
    }
    
    public function create(string $name, string $email, string $password): int {
        $stmt = $this->pdo->prepare(
            'INSERT INTO users (name, email, password_hash, created_at) 
             VALUES (:name, :email, :password, NOW())'
        );
        
        $stmt->execute([
            ':name' => $name,
            ':email' => $email,
            ':password' => password_hash($password, PASSWORD_DEFAULT)
        ]);
        
        return (int) $this->pdo->lastInsertId();
    }
    
    public function update(int $id, array $data): bool {
        $stmt = $this->pdo->prepare(
            'UPDATE users SET name = :name, email = :email WHERE id = :id'
        );
        
        return $stmt->execute([
            ':id' => $id,
            ':name' => $data['name'],
            ':email' => $data['email']
        ]);
    }
    
    public function delete(int $id): bool {
        $stmt = $this->pdo->prepare('DELETE FROM users WHERE id = :id');
        $stmt->execute([':id' => $id]);
        return $stmt->rowCount() > 0;
    }
}
?>
```


## Resources

- [Prepared Statements](https://www.php.net/manual/en/pdo.prepared-statements.php) — Official guide to PDO prepared statements
- [Database Security](https://www.php.net/manual/en/security.database.sql-injection.php) — Preventing SQL injection attacks

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*