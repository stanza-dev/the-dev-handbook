---
source_course: "php-databases"
source_lesson: "php-databases-pdo-configuration"
---

# PDO Configuration & Best Practices

## Introduction

PDO (PHP Data Objects) provides a consistent, database-agnostic interface for accessing relational databases in PHP. Proper configuration is the foundation of secure, reliable database interactions and prevents entire classes of bugs before they happen.

## Key Concepts

- **DSN (Data Source Name)**: A connection string specifying the driver, host, database name, and charset.
- **PDO Attributes**: Configuration flags that control PDO behavior such as error handling, fetch modes, and prepared statement emulation.
- **Driver-Specific Constants**: Constants like `Pdo\Mysql::ATTR_*` that configure driver-specific behavior.

## Real World Context

Every PHP application that talks to a database needs a properly configured PDO connection. Misconfigured connections are a leading cause of SQL injection vulnerabilities, silent data corruption, and hard-to-debug production issues. Getting this right from the start saves hours of debugging later.

## Deep Dive

The most important configuration decision is setting up your connection options correctly. Since PHP 8.0, `PDO::ERRMODE_EXCEPTION` is the default error mode, so you no longer need to set it explicitly. However, other critical options still need manual configuration.

```php
<?php
$dsn = 'mysql:host=localhost;dbname=myapp;charset=utf8mb4';
$options = [
    // No need to set ERRMODE_EXCEPTION — default since PHP 8.0
    
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
    error_log($e->getMessage());
    throw new RuntimeException('Database connection failed');
}
```

Disabling emulated prepares is critical for security. With real prepared statements, the query template is sent to the database first, then parameters are sent separately.

```php
<?php
// With emulated prepares: PHP builds the final SQL string
// With real prepares: query and data travel separately
$id = '1 OR 1=1';

// Emulated: potential type juggling attacks
// Real: database rejects non-integer for INT column
```

In PHP 8.5, driver-specific constants have been reorganized. The old constants are deprecated:

```php
<?php
// DEPRECATED in PHP 8.5:
PDO::MYSQL_ATTR_FOUND_ROWS => true,
PDO::PGSQL_ATTR_DISABLE_PREPARES => true,
PDO::SQLITE_ATTR_OPEN_FLAGS => SQLITE3_OPEN_READWRITE,

// PREFERRED in PHP 8.5:
Pdo\Mysql::ATTR_FOUND_ROWS => true,
Pdo\Pgsql::ATTR_DISABLE_PREPARES => true,
Pdo\Sqlite::ATTR_OPEN_FLAGS => SQLITE3_OPEN_READWRITE,
```

The `PDO "uri:" DSN scheme` is also deprecated in PHP 8.5. Pass the DSN string directly instead of loading from a file.

A database factory encapsulates connection creation:

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
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
            PDO::ATTR_EMULATE_PREPARES => false,
            Pdo\Mysql::ATTR_FOUND_ROWS => true,
        ];
    }
}
```

For PostgreSQL connections:

```php
<?php
$dsn = 'pgsql:host=localhost;dbname=myapp';
$pdo = new PDO($dsn, $username, $password, [
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
]);
$pdo->exec('SET search_path TO myschema');
```

## Common Pitfalls

1. **Setting ERRMODE_EXCEPTION explicitly on PHP 8.0+** — It is the default. While harmless, it clutters code and signals unawareness of modern defaults.
2. **Leaving EMULATE_PREPARES enabled** — Emulated prepares bypass the database's type enforcement, opening the door to type-juggling attacks.

## Best Practices

1. **Always disable emulated prepares** — Set `ATTR_EMULATE_PREPARES => false` so the database handles parameterization natively.
2. **Use the new driver subclass constants** — Replace `PDO::MYSQL_ATTR_*` with `Pdo\Mysql::ATTR_*` to avoid deprecation warnings in PHP 8.5.

## Summary

- PDO provides a unified interface for database access; configuration determines security and reliability.
- Since PHP 8.0, `ERRMODE_EXCEPTION` is the default and does not need to be set explicitly.
- PHP 8.5 deprecates `PDO::MYSQL_ATTR_*` in favor of `Pdo\Mysql::ATTR_*`, and the `uri:` DSN scheme.
- Always disable emulated prepares for real parameterized queries.

## Code Examples

**PHP 8.5 PDO configuration with driver subclass constants**

```php
<?php
declare(strict_types=1);

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
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
            PDO::ATTR_EMULATE_PREPARES => false,
            PDO::ATTR_STRINGIFY_FETCHES => false,
            Pdo\Mysql::ATTR_FOUND_ROWS => true,
        ];
    }
}
?>
```


## Resources

- [PDO Manual](https://www.php.net/manual/en/book.pdo.php) — Complete PDO documentation
- [PHP Deprecated Attribute RFC](https://wiki.php.net/rfc/deprecated_attribute) — RFC for the #[\Deprecated] attribute used to mark deprecated constants

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*