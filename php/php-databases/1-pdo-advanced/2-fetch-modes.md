---
source_course: "php-databases"
source_lesson: "php-databases-pdo-fetch-modes"
---

# PDO Fetch Modes & Hydration

## Introduction

PDO offers multiple strategies for retrieving and structuring query results. Choosing the right fetch mode eliminates boilerplate code and lets you map database rows directly into the data structures your application needs.

## Key Concepts

- **Fetch Mode**: A PDO constant that determines the format of returned rows (array, object, class instance).
- **Hydration**: The process of populating an object's properties from a database row.
- **FETCH_CLASS**: A fetch mode that instantiates a specified class and maps columns to its properties.

## Real World Context

In a real application, you rarely want raw associative arrays flowing through your business logic. Fetch modes let you go from database rows to typed domain objects in a single step, improving IDE support and reducing mapping code.

## Deep Dive

The most commonly used fetch modes each serve a distinct purpose:

```php
<?php
$stmt = $pdo->query('SELECT id, name, email FROM users');

// FETCH_ASSOC - Associative array (recommended default)
$user = $stmt->fetch(PDO::FETCH_ASSOC);
// ['id' => 1, 'name' => 'John', 'email' => 'john@example.com']

// FETCH_NUM - Numeric array
$user = $stmt->fetch(PDO::FETCH_NUM);
// [1, 'John', 'john@example.com']

// FETCH_OBJ - stdClass object
$user = $stmt->fetch(PDO::FETCH_OBJ);
// $user->id, $user->name, $user->email
```

For domain-driven code, `FETCH_CLASS` hydrates results directly into your classes:

```php
<?php
class User {
    public int $id;
    public string $name;
    public string $email;
    
    public function getDisplayName(): string {
        return $this->name;
    }
}

$stmt = $pdo->query('SELECT id, name, email FROM users');
$stmt->setFetchMode(PDO::FETCH_CLASS, User::class);

foreach ($stmt as $user) {
    echo $user->getDisplayName();
}
```

Use `FETCH_PROPS_LATE` when your class has a constructor that sets defaults:

```php
<?php
$stmt->setFetchMode(
    PDO::FETCH_CLASS | PDO::FETCH_PROPS_LATE,
    User::class,
    ['constructorArg1']
);
```

Bulk fetch operations provide powerful data restructuring:

```php
<?php
// Single column
$emails = $stmt->fetchAll(PDO::FETCH_COLUMN, 2);

// Key-value pairs
$stmt = $pdo->query('SELECT id, name FROM users');
$names = $stmt->fetchAll(PDO::FETCH_KEY_PAIR);
// [1 => 'John', 2 => 'Jane']

// Grouped by first column
$stmt = $pdo->query('SELECT role, id, name FROM users');
$byRole = $stmt->fetchAll(PDO::FETCH_GROUP | PDO::FETCH_ASSOC);

// Unique by first column
$indexed = $stmt->fetchAll(PDO::FETCH_UNIQUE | PDO::FETCH_ASSOC);
```

Custom hydration via callback:

```php
<?php
$users = $stmt->fetchAll(PDO::FETCH_FUNC, function($id, $name, $email) {
    return new User((int) $id, $name, $email);
});
```

## Common Pitfalls

1. **Using FETCH_BOTH as the default** — It doubles memory usage by storing each value under both numeric and string keys.
2. **Forgetting FETCH_PROPS_LATE with constructors** — Without this flag, PDO sets properties before calling the constructor, which can overwrite them.

## Best Practices

1. **Set FETCH_ASSOC as the connection default** — Configure it once in your PDO options so every query benefits.
2. **Use FETCH_KEY_PAIR for lookup tables** — Builds an id-to-name map in one call without manual looping.

## Summary

- PDO provides multiple fetch modes for different data structure needs.
- `FETCH_ASSOC` is the best default; `FETCH_CLASS` bridges the gap to domain objects.
- Bulk fetch variations like `FETCH_KEY_PAIR` and `FETCH_GROUP` eliminate manual array restructuring.
- Use `FETCH_PROPS_LATE` when hydrating classes that have constructors.

## Code Examples

**Repository with various fetch patterns**

```php
<?php
declare(strict_types=1);

class UserRepository {
    public function __construct(private PDO $pdo) {}
    
    public function findById(int $id): ?User {
        $stmt = $this->pdo->prepare(
            'SELECT id, name, email, created_at FROM users WHERE id = :id'
        );
        $stmt->execute(['id' => $id]);
        $stmt->setFetchMode(PDO::FETCH_CLASS, User::class);
        return $stmt->fetch() ?: null;
    }
    
    /** @return User[] */
    public function findAll(): array {
        $stmt = $this->pdo->query(
            'SELECT id, name, email, created_at FROM users ORDER BY name'
        );
        $stmt->setFetchMode(PDO::FETCH_CLASS, User::class);
        return $stmt->fetchAll();
    }
    
    /** @return array<int, string> */
    public function getEmailsById(): array {
        $stmt = $this->pdo->query('SELECT id, email FROM users');
        return $stmt->fetchAll(PDO::FETCH_KEY_PAIR);
    }
}
?>
```


## Resources

- [PDOStatement::fetch](https://www.php.net/manual/en/pdostatement.fetch.php) — PDO fetch modes documentation

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*