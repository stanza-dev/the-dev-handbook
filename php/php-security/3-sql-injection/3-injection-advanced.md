---
source_course: "php-security"
source_lesson: "php-security-sql-injection-advanced"
---

# Advanced SQL Injection Prevention

## Introduction
Beyond basic prepared statements, learn techniques for complex queries and edge cases.

## Key Concepts
- **Second-Order SQL Injection**: Malicious data stored safely, then later used unsafely in a different query.
- **Parameterized Identifiers**: Table/column names cannot be parameterized — use strict whitelist validation.
- **Emulated vs Native Prepares**: `ATTR_EMULATE_PREPARES = false` ensures the database truly separates query structure from data.
- **Stored Procedure Injection**: Even stored procedures can be vulnerable if they use dynamic SQL internally.

## Real World Context
Second-order SQL injection bypasses input sanitization because the malicious payload enters the database through a safe path (e.g., user registration) and triggers later (e.g., password reset). This is harder to detect with automated scanners and requires careful code review of all database queries.

## Deep Dive
### Intro

Beyond basic prepared statements, learn techniques for complex queries and edge cases.

### Dynamic column selection

```php
<?php
class SafeQueryBuilder
{
    private const ALLOWED_COLUMNS = [
        'users' => ['id', 'name', 'email', 'created_at', 'status'],
        'orders' => ['id', 'user_id', 'total', 'status', 'created_at'],
    ];
    
    private const ALLOWED_ORDER = ['ASC', 'DESC'];
    
    public function select(string $table, array $columns): string
    {
        // Validate table
        if (!isset(self::ALLOWED_COLUMNS[$table])) {
            throw new InvalidArgumentException('Invalid table');
        }
        
        // Validate columns
        $allowedCols = self::ALLOWED_COLUMNS[$table];
        $safeCols = array_intersect($columns, $allowedCols);
        
        if (empty($safeCols)) {
            $safeCols = $allowedCols;  // Default to all allowed
        }
        
        return sprintf(
            'SELECT %s FROM %s',
            implode(', ', array_map(fn($c) => "`$c`", $safeCols)),
            "`$table`"
        );
    }
    
    public function orderBy(string $column, string $direction): string
    {
        $direction = strtoupper($direction);
        
        if (!in_array($direction, self::ALLOWED_ORDER, true)) {
            $direction = 'ASC';
        }
        
        // Column must be validated against allowed list elsewhere
        return "ORDER BY `$column` $direction";
    }
}
```

### Safe in clause

```php
<?php
function findByIds(PDO $pdo, array $ids): array
{
    // Filter to integers only
    $ids = array_filter($ids, 'is_int');
    
    if (empty($ids)) {
        return [];
    }
    
    // Create placeholders
    $placeholders = implode(',', array_fill(0, count($ids), '?'));
    
    $stmt = $pdo->prepare("SELECT * FROM users WHERE id IN ($placeholders)");
    $stmt->execute(array_values($ids));
    
    return $stmt->fetchAll();
}

// Named parameter version
function findByIdsNamed(PDO $pdo, array $ids): array
{
    $params = [];
    $placeholders = [];
    
    foreach ($ids as $i => $id) {
        $key = ":id$i";
        $placeholders[] = $key;
        $params[$key] = $id;
    }
    
    $sql = 'SELECT * FROM users WHERE id IN (' . implode(',', $placeholders) . ')';
    $stmt = $pdo->prepare($sql);
    $stmt->execute($params);
    
    return $stmt->fetchAll();
}
```

### Safe like queries

```php
<?php
function searchUsers(PDO $pdo, string $term): array
{
    // Escape LIKE special characters
    $term = addcslashes($term, '%_\\');
    
    $stmt = $pdo->prepare(
        'SELECT * FROM users WHERE name LIKE :term ESCAPE "\\"'
    );
    $stmt->execute(['term' => '%' . $term . '%']);
    
    return $stmt->fetchAll();
}
```

### Preventing second-order sql injection

```php
<?php
// Second-order: Malicious data stored, then used unsafely later

// BAD: Data from database used without parameterization
$user = $pdo->query("SELECT * FROM users WHERE id = 1")->fetch();
$orders = $pdo->query("SELECT * FROM orders WHERE user_name = '{$user['name']}'");
// If user.name contains injection, it executes!

// GOOD: Always use prepared statements, even for "trusted" data
$stmt = $pdo->prepare('SELECT * FROM orders WHERE user_name = ?');
$stmt->execute([$user['name']]);
```

### Pdo security settings

```php
<?php
$pdo = new PDO($dsn, $user, $pass, [
    // Throw exceptions on errors
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    
    // Disable emulated prepares (use real prepared statements)
    PDO::ATTR_EMULATE_PREPARES => false,
    
    // Return integers as int, not string
    PDO::ATTR_STRINGIFY_FETCHES => false,
]);

// Why disable emulated prepares?
// Emulated: PHP escapes values, sends full query
// Real: Database receives query + parameters separately
// Real prepared statements are safer and can be reused
```

### Stored procedures (additional layer)

```sql
-- MySQL stored procedure
DELIMITER //
CREATE PROCEDURE GetUserByEmail(IN p_email VARCHAR(255))
BEGIN
    SELECT id, name, email FROM users WHERE email = p_email;
END //
DELIMITER ;
```

```php
<?php
// Call from PHP
$stmt = $pdo->prepare('CALL GetUserByEmail(:email)');
$stmt->execute(['email' => $email]);
$user = $stmt->fetch();
```

## Common Pitfalls
1. **Trusting data from your own database** — Data retrieved from the database may have been injected by a previous request. Always use prepared statements for ALL queries.
2. **Dynamic table/column names from user input** — Since identifiers can't be parameterized, unvalidated table names lead to SQL injection. Always whitelist allowed identifiers.

## Best Practices
1. **Disable emulated prepares** — Set `PDO::ATTR_EMULATE_PREPARES` to `false` so the database engine handles parameter binding natively.
2. **Use an ORM for dynamic queries** — ORMs like Doctrine or Eloquent handle parameterization and identifier escaping, reducing manual SQL injection risks.

## Summary
- Second-order SQL injection exploits data that was stored safely but used unsafely later.
- Always use native prepared statements with `ATTR_EMULATE_PREPARES = false`.
- Whitelist-validate all dynamic identifiers since they cannot be parameterized.

## Resources

- [PDO Prepared Statements](https://www.php.net/manual/en/pdo.prepared-statements.php) — PHP PDO prepared statements documentation

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*