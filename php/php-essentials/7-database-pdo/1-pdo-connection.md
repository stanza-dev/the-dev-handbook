---
source_course: "php-essentials"
source_lesson: "php-essentials-pdo-connection"
---

# Connecting to Databases with PDO

## Introduction

PDO (PHP Data Objects) is PHP's modern database abstraction layer, providing a consistent API for working with MySQL, PostgreSQL, SQLite, and other databases. Instead of learning a different set of functions for each database, PDO gives you one interface for all of them. This lesson covers establishing connections, configuring options, and structuring your database access code.

## Key Concepts

- **PDO (PHP Data Objects)**: A database abstraction layer that provides a uniform interface for multiple database systems.
- **DSN (Data Source Name)**: A connection string specifying the database driver, host, database name, and charset.
- **PDO attributes**: Configuration options (error mode, fetch mode, prepared statement emulation) that control PDO's behavior.
- **Singleton pattern**: A design pattern ensuring only one database connection instance exists throughout the application.

## Real World Context

Every PHP application that stores data needs a database connection. Before PDO, developers used database-specific functions like `mysql_query()` that locked them into a single database vendor. PDO lets you switch from MySQL to PostgreSQL by changing one line (the DSN). More importantly, PDO's prepared statements are the industry-standard defense against SQL injection, the most common web security vulnerability.

## Deep Dive

### Why PDO?

PDO offers four major advantages:

- **Security**: Built-in prepared statements prevent SQL injection.
- **Portability**: The same code works with MySQL, PostgreSQL, SQLite, and more.
- **Object-Oriented**: A modern, clean API with proper exception handling.
- **Error Handling**: Configurable error modes including exceptions.

### Basic Connection

The simplest PDO connection wraps the constructor in a try-catch:

```php
<?php
try {
    $pdo = new PDO(
        'mysql:host=localhost;dbname=myapp;charset=utf8mb4',
        'username',
        'password'
    );
} catch (PDOException $e) {
    die("Connection failed: " . $e->getMessage());
}
```

The first argument is the DSN, followed by the username and password.

### Connection with Options (Recommended)

Always set these three options for production use:

```php
<?php
$dsn = 'mysql:host=localhost;dbname=myapp;charset=utf8mb4';
$username = 'root';
$password = 'secret';

$options = [
    // Throw exceptions on errors
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    // Return associative arrays by default
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    // Don't emulate prepared statements
    PDO::ATTR_EMULATE_PREPARES => false,
];

try {
    $pdo = new PDO($dsn, $username, $password, $options);
} catch (PDOException $e) {
    throw new PDOException($e->getMessage(), (int) $e->getCode());
}
```

`ERRMODE_EXCEPTION` is critical — without it, PDO silently returns `false` on errors. `EMULATE_PREPARES => false` ensures the database driver handles prepared statements natively, which is both safer and more efficient.

### DSN Formats for Different Databases

The DSN format varies by database driver:

```php
<?php
// MySQL / MariaDB
$dsn = 'mysql:host=localhost;dbname=myapp;charset=utf8mb4';

// PostgreSQL
$dsn = 'pgsql:host=localhost;dbname=myapp';

// SQLite (file-based)
$dsn = 'sqlite:/path/to/database.db';

// SQLite (in-memory)
$dsn = 'sqlite::memory:';
```

This is the only line you need to change when switching databases. The rest of your PDO code stays the same.

### Connection Class Pattern

A singleton pattern ensures your application reuses a single database connection:

```php
<?php
class Database {
    private static ?PDO $instance = null;
    
    public static function getConnection(): PDO {
        if (self::$instance === null) {
            $dsn = 'mysql:host=localhost;dbname=app;charset=utf8mb4';
            self::$instance = new PDO($dsn, 'user', 'pass', [
                PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
            ]);
        }
        return self::$instance;
    }
}

// Usage
$pdo = Database::getConnection();
```

This avoids opening multiple connections per request, which wastes resources and can hit connection limits.

## Common Pitfalls

1. **Not setting ERRMODE_EXCEPTION** — Without it, PDO fails silently and you get mysterious `false` return values instead of clear error messages.
2. **Hardcoding credentials in source code** — Use environment variables (`$_ENV['DB_HOST']`) or a `.env` file. Never commit database passwords to version control.
3. **Leaving emulate_prepares enabled** — Emulated prepares happen in PHP, not in the database, which can be less secure. Set `ATTR_EMULATE_PREPARES => false`.

## Best Practices

1. **Always pass options in the constructor** — Set error mode, fetch mode, and emulate_prepares at connection time so every query benefits from them.
2. **Use environment variables for credentials** — Load host, database, username, and password from environment variables or configuration files outside the web root.
3. **Catch `PDOException` specifically** — Catch `PDOException` rather than generic `Exception` to handle database errors separately from application errors.

## Summary

- PDO provides a unified API for MySQL, PostgreSQL, SQLite, and other databases.
- Always configure `ERRMODE_EXCEPTION`, `FETCH_ASSOC`, and `EMULATE_PREPARES => false`.
- The DSN string is the only thing that changes when switching database systems.
- Use a singleton or dependency injection pattern to reuse a single connection.
- Never hardcode database credentials; use environment variables.

## Code Examples

**Production database connection class using named arguments, environment variables, and all recommended PDO options**

```php
<?php
// Production-ready database connection class
declare(strict_types=1);

class Database {
    private PDO $pdo;
    
    public function __construct(
        string $host = 'localhost',
        string $database = '',
        string $username = '',
        string $password = '',
        int $port = 3306
    ) {
        $dsn = sprintf(
            'mysql:host=%s;port=%d;dbname=%s;charset=utf8mb4',
            $host,
            $port,
            $database
        );
        
        $options = [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
            PDO::ATTR_EMULATE_PREPARES => false,
            PDO::MYSQL_ATTR_INIT_COMMAND => "SET NAMES utf8mb4"
        ];
        
        $this->pdo = new PDO($dsn, $username, $password, $options);
    }
    
    public function getConnection(): PDO {
        return $this->pdo;
    }
}

// Usage with environment variables
$db = new Database(
    host: $_ENV['DB_HOST'] ?? 'localhost',
    database: $_ENV['DB_NAME'] ?? 'myapp',
    username: $_ENV['DB_USER'] ?? 'root',
    password: $_ENV['DB_PASS'] ?? ''
);
?>
```


## Resources

- [PDO Introduction](https://www.php.net/manual/en/intro.pdo.php) — Official PDO introduction and overview
- [PDO Connections](https://www.php.net/manual/en/pdo.connections.php) — Complete guide to PDO connection handling

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*