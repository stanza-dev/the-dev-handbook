---
source_course: "php-essentials"
source_lesson: "php-essentials-fetching-data"
---

# Fetching Query Results

PDO provides multiple ways to fetch data from the database. Choose the right method for your needs.

## fetch() - Single Row

```php
<?php
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$stmt->execute([':id' => 1]);

$user = $stmt->fetch();
// Returns one row or false if not found

if ($user) {
    echo $user['name'];
}
```

## fetchAll() - All Rows

```php
<?php
$stmt = $pdo->query('SELECT * FROM products WHERE active = 1');
$products = $stmt->fetchAll();

foreach ($products as $product) {
    echo $product['name'] . ': $' . $product['price'] . "\n";
}
```

## Fetch Modes

```php
<?php
// Associative array (most common)
$row = $stmt->fetch(PDO::FETCH_ASSOC);
// ['id' => 1, 'name' => 'John']

// Indexed array
$row = $stmt->fetch(PDO::FETCH_NUM);
// [0 => 1, 1 => 'John']

// Both associative and indexed
$row = $stmt->fetch(PDO::FETCH_BOTH);
// ['id' => 1, 0 => 1, 'name' => 'John', 1 => 'John']

// Object (stdClass)
$row = $stmt->fetch(PDO::FETCH_OBJ);
// stdClass { id: 1, name: 'John' }

// Fetch into a specific class
$user = $stmt->fetch(PDO::FETCH_CLASS, User::class);
```

## fetchColumn() - Single Value

```php
<?php
$stmt = $pdo->query('SELECT COUNT(*) FROM users');
$count = $stmt->fetchColumn();
echo "Total users: $count";

// Get specific column (0-indexed)
$stmt = $pdo->query('SELECT id, name, email FROM users LIMIT 1');
$email = $stmt->fetchColumn(2);  // Third column (email)
```

## Iterating Results

```php
<?php
$stmt = $pdo->query('SELECT * FROM users');

// Memory efficient - fetches one row at a time
while ($user = $stmt->fetch()) {
    echo $user['name'] . "\n";
}

// Or use foreach directly
foreach ($stmt as $user) {
    echo $user['name'] . "\n";
}
```

## Fetching into Objects

```php
<?php
class User {
    public int $id;
    public string $name;
    public string $email;
}

$stmt = $pdo->query('SELECT id, name, email FROM users');
$stmt->setFetchMode(PDO::FETCH_CLASS, User::class);

foreach ($stmt as $user) {
    // $user is a User instance
    echo $user->name;
}

// Or fetch all at once
$users = $stmt->fetchAll(PDO::FETCH_CLASS, User::class);
```

## Practical Tips

```php
<?php
// Check if results exist
$stmt->execute();
if ($stmt->rowCount() > 0) {
    // Has results
}

// Or simply
if ($user = $stmt->fetch()) {
    // Found
} else {
    // Not found
}

// Get column count
$columnCount = $stmt->columnCount();
```

## Code Examples

**Various data fetching strategies**

```php
<?php
// Different fetching strategies

// 1. Paginated listing
function getUsers(PDO $pdo, int $page = 1, int $perPage = 10): array {
    $offset = ($page - 1) * $perPage;
    
    $stmt = $pdo->prepare(
        'SELECT * FROM users ORDER BY created_at DESC LIMIT :limit OFFSET :offset'
    );
    $stmt->bindValue(':limit', $perPage, PDO::PARAM_INT);
    $stmt->bindValue(':offset', $offset, PDO::PARAM_INT);
    $stmt->execute();
    
    return $stmt->fetchAll();
}

// 2. Key-value pairs (useful for dropdowns)
function getCategoriesForDropdown(PDO $pdo): array {
    $stmt = $pdo->query('SELECT id, name FROM categories ORDER BY name');
    return $stmt->fetchAll(PDO::FETCH_KEY_PAIR);
    // Returns: [1 => 'Electronics', 2 => 'Clothing', ...]
}

// 3. Grouped data
function getUsersByRole(PDO $pdo): array {
    $stmt = $pdo->query('SELECT role, name FROM users ORDER BY role, name');
    return $stmt->fetchAll(PDO::FETCH_GROUP | PDO::FETCH_COLUMN);
    // Returns: ['admin' => ['Alice', 'Bob'], 'user' => ['Charlie', ...]]
}

// 4. Single record with null handling
function findUserOrFail(PDO $pdo, int $id): array {
    $stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
    $stmt->execute([':id' => $id]);
    
    $user = $stmt->fetch();
    
    if (!$user) {
        throw new RuntimeException("User not found: $id");
    }
    
    return $user;
}
?>
```


## Resources

- [PDOStatement::fetch](https://www.php.net/manual/en/pdostatement.fetch.php) — Fetch method documentation and modes

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*