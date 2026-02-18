---
source_course: "php-databases"
source_lesson: "php-databases-pdo-configuration"
---

# PDO Configuration & Best Practices

PDO (PHP Data Objects) provides a consistent interface for database access. Proper configuration is essential for security and reliability.

## Connection Setup

```php
<?php
$dsn = 'mysql:host=localhost;dbname=myapp;charset=utf8mb4';
$options = [
    // Throw exceptions on errors
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    
    // Return associative arrays by default
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    
    // Use real prepared statements (not emulated)
    PDO::ATTR_EMULATE_PREPARES => false,
    
    // Return strings as PHP strings, not LOBs
    PDO::ATTR_STRINGIFY_FETCHES => false,
];

try {
    $pdo = new PDO($dsn, $username, $password, $options);
} catch (PDOException $e) {
    // Don't expose connection details
    error_log($e->getMessage());
    throw new RuntimeException('Database connection failed');
}
```

## Why ATTR_EMULATE_PREPARES Should Be False

```php
<?php
// With emulated prepares (default)
// PHP builds the query, then sends it as one string
// Less secure, potential type issues

// With real prepares
// Query template sent first, then parameters separately
// Database enforces types, better security

// Example: Integer handling
$id = '1 OR 1=1';

// Emulated: May allow type juggling attacks
// Real: Database rejects non-integer for INT column
```

## Database Factory

```php
<?php
class DatabaseFactory {
    private static ?PDO $instance = null;
    
    public static function create(): PDO {
        if (self::$instance === null) {
            $config = require 'config/database.php';
            
            $dsn = sprintf(
                '%s:host=%s;port=%d;dbname=%s;charset=utf8mb4',
                $config['driver'],
                $config['host'],
                $config['port'],
                $config['database']
            );
            
            self::$instance = new PDO(
                $dsn,
                $config['username'],
                $config['password'],
                self::getOptions()
            );
        }
        
        return self::$instance;
    }
    
    private static function getOptions(): array {
        return [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
            PDO::ATTR_EMULATE_PREPARES => false,
            PDO::MYSQL_ATTR_FOUND_ROWS => true,
        ];
    }
}
```

## PostgreSQL Connection

```php
<?php
$dsn = 'pgsql:host=localhost;dbname=myapp';
$options = [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
];

$pdo = new PDO($dsn, $username, $password, $options);

// PostgreSQL-specific: Set schema
$pdo->exec('SET search_path TO myschema');
```

## Resources

- [PDO Manual](https://www.php.net/manual/en/book.pdo.php) — Complete PDO documentation

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*