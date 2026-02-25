---
source_course: "php-security"
source_lesson: "php-security-prepared-statements"
---

# Prepared Statements: The Solution

## Introduction
Prepared statements completely prevent SQL injection by separating SQL code from data.

## Key Concepts
- **Query Template**: SQL structure is sent to the database first, separate from data values.
- **Parameter Binding**: Data values are sent separately and never interpreted as SQL code.
- **Named Parameters**: Use `:name` placeholders for readable, maintainable prepared statements.
- **Identifier Limitation**: Table and column names cannot be parameterized — use whitelist validation instead.

## Real World Context
Prepared statements are the industry-standard defense against SQL injection, recommended by OWASP, PHP documentation, and every major security framework. Companies that suffered SQL injection breaches (like Sony in 2011) were consistently found to be using string concatenation instead of prepared statements.

## Deep Dive
### Intro

Prepared statements completely prevent SQL injection by separating SQL code from data.

### How prepared statements work

```
1. PREPARE: Send SQL template to database
   "SELECT * FROM users WHERE email = ?"

2. BIND: Send data separately
   Data: "user@example.com"

3. EXECUTE: Database combines them safely
   The data is NEVER interpreted as SQL
```

### Pdo prepared statements

### Named Parameters

```php
<?php
$email = $_POST['email'];  // Even if malicious

$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => $email]);
$user = $stmt->fetch();

// The malicious input "'; DROP TABLE users;--"
// is treated as a LITERAL STRING, not SQL!
```

### Positional Parameters

```php
<?php
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = ? AND status = ?');
$stmt->execute([$email, 'active']);
```

### INSERT with Prepared Statements

```php
<?php
$stmt = $pdo->prepare(
    'INSERT INTO users (name, email, password_hash) VALUES (:name, :email, :password)'
);

$stmt->execute([
    'name' => $name,
    'email' => $email,
    'password' => password_hash($password, PASSWORD_DEFAULT)
]);
```

### What you cannot parameterize

Parameters only work for DATA, not SQL structure:

```php
<?php
// WRONG: Can't parameterize table/column names
$stmt = $pdo->prepare('SELECT * FROM :table');  // Error!
$stmt = $pdo->prepare('SELECT * FROM users ORDER BY :column');  // Error!

// RIGHT: Whitelist for dynamic SQL parts
$allowedColumns = ['name', 'email', 'created_at'];
$orderBy = in_array($_GET['sort'], $allowedColumns, true) 
    ? $_GET['sort'] 
    : 'created_at';

$stmt = $pdo->prepare("SELECT * FROM users ORDER BY $orderBy");
```

## Common Pitfalls
```php
<?php
// WRONG: Concatenating user input
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = " . $_GET['id']);

// WRONG: Using query() with user input
$pdo->query("SELECT * FROM users WHERE id = {$_GET['id']}");

// WRONG: Prepared but not using parameters
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = '$id'");

// RIGHT: Always use parameters for user data
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$stmt->execute(['id' => $_GET['id']]);
```

## Best Practices
1. **Disable emulated prepares** — Set `PDO::ATTR_EMULATE_PREPARES` to `false` so the database engine handles parameter binding natively.
2. **Use named parameters for complex queries** — Named parameters (`:email`) are more readable than positional ones (`?`) and reduce binding errors in queries with many parameters.

## Summary
- Prepared statements prevent SQL injection by completely separating SQL structure from data.
- Always use parameter binding for user-supplied values in WHERE, INSERT, and UPDATE clauses.
- Table and column names cannot be parameterized — validate them against a strict whitelist.

## Code Examples

**Secure database wrapper with validated identifiers**

```php
<?php
declare(strict_types=1);

// Secure database wrapper
class SecureDatabase {
    public function __construct(
        private PDO $pdo
    ) {
        // Ensure PDO throws exceptions
        $this->pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
        // Disable emulated prepares for real server-side preparation
        $this->pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
    }
    
    public function findOne(string $table, array $conditions): ?array {
        $this->validateTableName($table);
        
        $where = [];
        foreach (array_keys($conditions) as $column) {
            $this->validateColumnName($column);
            $where[] = "$column = :$column";
        }
        
        $sql = sprintf(
            'SELECT * FROM %s WHERE %s LIMIT 1',
            $table,
            implode(' AND ', $where)
        );
        
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute($conditions);
        
        return $stmt->fetch(PDO::FETCH_ASSOC) ?: null;
    }
    
    public function insert(string $table, array $data): int {
        $this->validateTableName($table);
        
        $columns = [];
        $placeholders = [];
        
        foreach (array_keys($data) as $column) {
            $this->validateColumnName($column);
            $columns[] = $column;
            $placeholders[] = ":$column";
        }
        
        $sql = sprintf(
            'INSERT INTO %s (%s) VALUES (%s)',
            $table,
            implode(', ', $columns),
            implode(', ', $placeholders)
        );
        
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute($data);
        
        return (int) $this->pdo->lastInsertId();
    }
    
    private function validateTableName(string $table): void {
        if (!preg_match('/^[a-zA-Z_][a-zA-Z0-9_]*$/', $table)) {
            throw new InvalidArgumentException('Invalid table name');
        }
    }
    
    private function validateColumnName(string $column): void {
        if (!preg_match('/^[a-zA-Z_][a-zA-Z0-9_]*$/', $column)) {
            throw new InvalidArgumentException('Invalid column name');
        }
    }
}
?>
```


## Resources

- [PDO Prepared Statements](https://www.php.net/manual/en/pdo.prepared-statements.php) — PHP PDO prepared statements documentation

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*