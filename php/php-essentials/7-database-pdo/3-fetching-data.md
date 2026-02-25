---
source_course: "php-essentials"
source_lesson: "php-essentials-fetching-data"
---

# Fetching Query Results

## Introduction

Once you have executed a query, you need to retrieve the results. PDO provides multiple fetch methods and modes, each suited to different scenarios. Choosing the right one affects both the structure of your data and the memory efficiency of your application. This lesson covers all the major fetching strategies.

## Key Concepts

- **`fetch()`**: Retrieves a single row from the result set. Returns `false` when no more rows are available.
- **`fetchAll()`**: Retrieves all rows at once as an array. Convenient but uses more memory.
- **`fetchColumn()`**: Retrieves a single value from the next row, useful for COUNT queries.
- **Fetch mode**: Controls how each row is represented — as an associative array, numeric array, object, or class instance.
- **`PDO::FETCH_ASSOC`**: Returns rows as associative arrays with column names as keys (the most common mode).

## Real World Context

In production applications, you fetch data in different ways depending on the context: a single user record for a profile page, a paginated list of products for a catalog, a count for a dashboard metric, or key-value pairs for a dropdown menu. Knowing which fetch method to use in each situation keeps your code clean and performant. Using `fetchAll()` on a million-row table would exhaust memory, while using `fetch()` in a loop processes rows one at a time.

## Deep Dive

### fetch() — Single Row

Use `fetch()` when you expect exactly one result:

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

`fetch()` returns `false` when no row matches, so a simple `if` check handles the not-found case.

### fetchAll() — All Rows

Use `fetchAll()` for result sets you want to process as a complete array:

```php
<?php
$stmt = $pdo->query('SELECT * FROM products WHERE active = 1');
$products = $stmt->fetchAll();

foreach ($products as $product) {
    echo $product['name'] . ': $' . $product['price'] . "\n";
}
```

Be mindful of memory: `fetchAll()` loads every row into memory at once. For large result sets, use `fetch()` in a loop instead.

### Fetch Modes

PDO supports several fetch modes that control the data structure:

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

`FETCH_ASSOC` is the most widely used because column names serve as self-documenting keys.

### fetchColumn() — Single Value

Perfect for aggregate queries like COUNT, SUM, or MAX:

```php
<?php
$stmt = $pdo->query('SELECT COUNT(*) FROM users');
$count = $stmt->fetchColumn();
echo "Total users: $count";

// Get a specific column (0-indexed)
$stmt = $pdo->query('SELECT id, name, email FROM users LIMIT 1');
$email = $stmt->fetchColumn(2);  // Third column (email)
```

`fetchColumn()` without arguments returns the first column.

### Iterating Results Efficiently

For large result sets, iterate with `fetch()` instead of loading everything with `fetchAll()`:

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

Both approaches process one row at a time, keeping memory usage constant regardless of result set size.

### Fetching into Objects

Map rows directly to class instances for object-oriented code:

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

This is a lightweight alternative to a full ORM when you just need typed objects.

## Common Pitfalls

1. **Using `fetchAll()` on large result sets** — Loading 100,000 rows into memory at once will exhaust PHP's memory limit. Use `fetch()` in a loop for large datasets.
2. **Forgetting that `fetch()` returns `false`, not `null`** — Check with `if ($row)` or `if ($row !== false)`. Using strict null checks (`=== null`) will not catch the no-result case.
3. **Not setting a default fetch mode** — Without `PDO::ATTR_DEFAULT_FETCH_MODE`, PDO defaults to `FETCH_BOTH`, which doubles memory usage by returning both named and numeric keys.

## Best Practices

1. **Set `FETCH_ASSOC` as the default mode** — Configure it in the PDO constructor options so you never need to pass it to individual fetch calls.
2. **Use `fetchColumn()` for single values** — It is cleaner than fetching a full row just to read one field.
3. **Use `FETCH_CLASS` for domain objects** — When your application uses typed classes, map database rows directly to objects for type safety and IDE autocompletion.

## Summary

- `fetch()` returns one row; `fetchAll()` returns all rows; `fetchColumn()` returns a single value.
- `PDO::FETCH_ASSOC` is the most practical mode, returning rows as column-name-keyed arrays.
- Use `fetch()` in a loop for large result sets to avoid memory exhaustion.
- `FETCH_CLASS` maps rows directly to PHP class instances.
- Always set the default fetch mode in the PDO constructor options.

## Code Examples

**Four common fetching patterns: pagination, dropdown key-value pairs, grouped data, and single-record with error handling**

```php
<?php
// Different fetching strategies for common scenarios

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

// 2. Key-value pairs for dropdowns
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

// 4. Single record with error handling
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