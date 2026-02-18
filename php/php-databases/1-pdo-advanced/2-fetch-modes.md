---
source_course: "php-databases"
source_lesson: "php-databases-pdo-fetch-modes"
---

# PDO Fetch Modes & Hydration

PDO offers multiple ways to retrieve data. Choose the right fetch mode for your use case.

## Fetch Modes

```php
<?php
$stmt = $pdo->query('SELECT id, name, email FROM users');

// FETCH_ASSOC - Associative array (recommended)
$user = $stmt->fetch(PDO::FETCH_ASSOC);
// ['id' => 1, 'name' => 'John', 'email' => 'john@example.com']

// FETCH_NUM - Numeric array
$user = $stmt->fetch(PDO::FETCH_NUM);
// [1, 'John', 'john@example.com']

// FETCH_BOTH - Both keys (default, wasteful)
$user = $stmt->fetch(PDO::FETCH_BOTH);
// ['id' => 1, 0 => 1, 'name' => 'John', 1 => 'John', ...]

// FETCH_OBJ - stdClass object
$user = $stmt->fetch(PDO::FETCH_OBJ);
// object with $user->id, $user->name, $user->email
```

## Fetching Into Objects

```php
<?php
class User {
    public int $id;
    public string $name;
    public string $email;
    private ?string $passwordHash = null;
    
    public function getDisplayName(): string {
        return $this->name;
    }
}

// FETCH_CLASS - Hydrate into class
$stmt = $pdo->query('SELECT id, name, email FROM users');
$stmt->setFetchMode(PDO::FETCH_CLASS, User::class);

foreach ($stmt as $user) {
    echo $user->getDisplayName();  // Method available!
}

// With constructor arguments
$stmt->setFetchMode(
    PDO::FETCH_CLASS | PDO::FETCH_PROPS_LATE,
    User::class,
    ['constructorArg1', 'constructorArg2']
);
```

## Fetch All Variations

```php
<?php
// All rows as array of associative arrays
$users = $stmt->fetchAll(PDO::FETCH_ASSOC);

// Single column from all rows
$emails = $stmt->fetchAll(PDO::FETCH_COLUMN, 2);  // Column index
// ['john@example.com', 'jane@example.com', ...]

// Key-value pairs (first column as key)
$stmt = $pdo->query('SELECT id, name FROM users');
$names = $stmt->fetchAll(PDO::FETCH_KEY_PAIR);
// [1 => 'John', 2 => 'Jane', ...]

// Grouped by first column
$stmt = $pdo->query('SELECT role, id, name FROM users');
$byRole = $stmt->fetchAll(PDO::FETCH_GROUP | PDO::FETCH_ASSOC);
// ['admin' => [['id' => 1, 'name' => 'John']], 'user' => [...]]

// Unique by first column
$stmt = $pdo->query('SELECT id, name, email FROM users');
$indexed = $stmt->fetchAll(PDO::FETCH_UNIQUE | PDO::FETCH_ASSOC);
// [1 => ['name' => 'John', 'email' => '...'], 2 => [...]]
```

## Custom Hydration

```php
<?php
// FETCH_FUNC - Custom callback
$stmt = $pdo->query('SELECT id, name, email FROM users');
$users = $stmt->fetchAll(PDO::FETCH_FUNC, function($id, $name, $email) {
    return new User($id, $name, $email);
});
```

## Code Examples

**Repository with various fetch patterns**

```php
<?php
declare(strict_types=1);

// Repository with typed fetch methods
class UserRepository {
    public function __construct(
        private PDO $pdo
    ) {}
    
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
    
    /** @return array<string, User[]> */
    public function groupByStatus(): array {
        $stmt = $this->pdo->query(
            'SELECT status, id, name, email, created_at FROM users ORDER BY status, name'
        );
        
        $grouped = [];
        while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
            $status = $row['status'];
            unset($row['status']);
            $grouped[$status][] = $this->hydrate($row);
        }
        
        return $grouped;
    }
    
    private function hydrate(array $data): User {
        $user = new User();
        $user->id = (int) $data['id'];
        $user->name = $data['name'];
        $user->email = $data['email'];
        $user->createdAt = new DateTimeImmutable($data['created_at']);
        return $user;
    }
}
?>
```


## Resources

- [PDOStatement::fetch](https://www.php.net/manual/en/pdostatement.fetch.php) — PDO fetch modes documentation

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*