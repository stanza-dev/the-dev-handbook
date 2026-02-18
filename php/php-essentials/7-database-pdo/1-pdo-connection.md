---
source_course: "php-essentials"
source_lesson: "php-essentials-pdo-connection"
---

# Connecting to Databases with PDO

PDO (PHP Data Objects) is PHP's modern database abstraction layer. It provides a consistent interface for accessing different databases.

## Why PDO?

- **Security**: Built-in prepared statements prevent SQL injection
- **Portability**: Same code works with MySQL, PostgreSQL, SQLite, etc.
- **Object-Oriented**: Modern, clean API
- **Error Handling**: Proper exceptions instead of warnings

## Basic Connection

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

## Connection with Options (Recommended)

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

## DSN Formats for Different Databases

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

## Connection Class Pattern

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

## Code Examples

**Production database connection class**

```php
<?php
// Production-ready database connection
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