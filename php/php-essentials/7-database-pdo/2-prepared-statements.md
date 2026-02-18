---
source_course: "php-essentials"
source_lesson: "php-essentials-prepared-statements"
---

# Prepared Statements

Prepared statements are the **only safe way** to execute queries with user data. They completely prevent SQL injection.

## How Prepared Statements Work

1. **Prepare**: Send query template with placeholders
2. **Bind/Execute**: Send data separately
3. **Fetch**: Retrieve results

The database knows what's SQL and what's data - they never mix!

## Named Parameters (Recommended)

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

## Positional Parameters

```php
<?php
$stmt = $pdo->prepare(
    'SELECT * FROM products WHERE price > ? AND category = ?'
);

$stmt->execute([29.99, 'electronics']);
$products = $stmt->fetchAll();
```

## INSERT Operations

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

## UPDATE Operations

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

## DELETE Operations

```php
<?php
$stmt = $pdo->prepare('DELETE FROM users WHERE id = :id');
$stmt->execute([':id' => $userId]);

if ($stmt->rowCount() > 0) {
    echo "User deleted successfully";
}
```

## Why Prepared Statements Prevent SQL Injection

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

## Code Examples

**Repository pattern with prepared statements**

```php
<?php
// Complete CRUD operations example
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