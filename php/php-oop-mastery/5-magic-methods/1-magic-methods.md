---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-magic-methods"
---

# Magic Methods

## Introduction
Magic methods are special methods that PHP calls automatically in certain situations. They all start with double underscores and enable powerful metaprogramming patterns like property overloading, string conversion, and custom serialization.

## Key Concepts
- **Constructor/Destructor**: `__construct()` initializes, `__destruct()` cleans up.
- **Property Overloading**: `__get()`, `__set()`, `__isset()`, `__unset()` intercept property access.
- **Method Overloading**: `__call()` and `__callStatic()` intercept method calls.
- **Serialization**: `__serialize()`/`__unserialize()` control how objects are serialized (replaces `__sleep`/`__wakeup`).

## Real World Context
Frameworks use magic methods extensively. ORMs use `__get()` and `__set()` for dynamic properties. Query builders use `__call()` for fluent APIs like `$query->whereStatus('active')`. Understanding magic methods lets you build similar expressive APIs.

## Deep Dive

### Constructor & Destructor

```php
<?php
class DatabaseConnection {
    private PDO $connection;

    public function __construct(string $dsn) {
        echo "Opening connection\n";
        $this->connection = new PDO($dsn);
    }

    public function __destruct() {
        echo "Closing connection\n";
    }
}
```

The destructor runs when the object is garbage collected or the script ends.

### Property Overloading

```php
<?php
class DynamicObject {
    private array $data = [];

    public function __get(string $name): mixed {
        return $this->data[$name] ?? null;
    }

    public function __set(string $name, mixed $value): void {
        $this->data[$name] = $value;
    }

    public function __isset(string $name): bool {
        return isset($this->data[$name]);
    }

    public function __unset(string $name): void {
        unset($this->data[$name]);
    }
}

$obj = new DynamicObject();
$obj->name = 'John';          // Calls __set
echo $obj->name;               // Calls __get
var_dump(isset($obj->name));   // Calls __isset
unset($obj->name);             // Calls __unset
```

These methods intercept access to undefined or inaccessible properties.

### Method Overloading

```php
<?php
class ApiClient {
    public function __call(string $method, array $args): mixed {
        if (str_starts_with($method, 'get')) {
            $endpoint = strtolower(substr($method, 3));
            return $this->request('GET', "/$endpoint", $args[0] ?? []);
        }
        throw new BadMethodCallException("Method $method not found");
    }

    public static function __callStatic(string $method, array $args): mixed {
        return (new self())->$method(...$args);
    }

    private function request(string $method, string $endpoint, array $params): array {
        return ['method' => $method, 'endpoint' => $endpoint];
    }
}

$client = new ApiClient();
$users = $client->getUsers();   // GET /users
$posts = ApiClient::getPosts(); // Static call
```

`__call()` and `__callStatic()` intercept calls to undefined methods.

### String Conversion and Cloning

```php
<?php
class Money {
    public function __construct(
        private int $cents,
        private string $currency = 'USD'
    ) {}

    public function __toString(): string {
        return sprintf('%s %.2f', $this->currency, $this->cents / 100);
    }
}

$price = new Money(1999);
echo $price;  // "USD 19.99"
```

`__toString()` is called whenever the object is used in a string context.

### Cloning with Clone With (PHP 8.5)

PHP 8.5 introduces a new `clone()` function syntax that allows modifying properties during cloning.

```php
<?php
class Order {
    public function __construct(
        public int $id,
        public array $items,
        public DateTimeImmutable $createdAt
    ) {}

    public function __clone(): void {
        $this->createdAt = clone $this->createdAt;
    }
}

$order1 = new Order(1, ['item1'], new DateTimeImmutable());

// Traditional clone
$order2 = clone $order1;
$order2->id = 0;

// PHP 8.5 Clone With syntax
$order3 = clone($order1, ['id' => 0, 'items' => ['item2']]);
// $order3->id is 0 and items is ['item2'], createdAt is deep cloned
```

Clone With provides a cleaner way to create modified copies of objects.

### Serialization (__serialize/__unserialize)

In PHP 8.5, `__sleep()` and `__wakeup()` are soft-deprecated. Use `__serialize()` and `__unserialize()` instead.

```php
<?php
class User {
    public function __construct(
        public string $name,
        public string $password
    ) {}

    // Modern approach (preferred since PHP 7.4, required going forward)
    public function __serialize(): array {
        return ['name' => $this->name]; // Exclude password
    }

    public function __unserialize(array $data): void {
        $this->name = $data['name'];
        $this->password = '';
    }

    // __sleep()/__wakeup() are soft-deprecated in PHP 8.5
    // Avoid using them in new code
}
```

`__serialize()` returns an array of data to serialize, giving you full control. `__unserialize()` receives that array and rebuilds the object.

## Common Pitfalls
1. **Overusing __get/__set** — Dynamic properties make code hard to understand and break IDE autocompletion. Prefer explicit properties.
2. **Performance overhead** — Magic methods are slower than regular methods. Do not use them in hot loops.

## Best Practices
1. **Use __serialize over __sleep** — The newer `__serialize()`/`__unserialize()` API is clearer and more powerful.
2. **Document dynamic behavior** — If you use `__call` or `__get`, add PHPDoc annotations so IDEs can provide autocompletion.

## Summary
- Magic methods are called automatically by PHP in specific situations.
- Property overloading intercepts access to undefined/inaccessible properties.
- PHP 8.5 introduces Clone With syntax and soft-deprecates `__sleep`/`__wakeup`.
- Use `__serialize()`/`__unserialize()` for modern serialization control.

## Code Examples

**Fluent query builder with magic methods**

```php
<?php
declare(strict_types=1);

class QueryBuilder {
    private string $table = '';
    private array $wheres = [];
    private array $selects = ['*'];
    private ?int $limit = null;

    public function __call(string $method, array $args): self {
        if (str_starts_with($method, 'where')) {
            $column = strtolower(substr($method, 5));
            return $this->where($column, $args[0]);
        }
        throw new BadMethodCallException("Method $method not found");
    }

    public function table(string $table): self {
        $this->table = $table;
        return $this;
    }

    public function select(string ...$columns): self {
        $this->selects = $columns;
        return $this;
    }

    public function where(string $column, mixed $value): self {
        $this->wheres[] = [$column, $value];
        return $this;
    }

    public function limit(int $limit): self {
        $this->limit = $limit;
        return $this;
    }

    public function __toString(): string {
        $sql = 'SELECT ' . implode(', ', $this->selects);
        $sql .= ' FROM ' . $this->table;
        if ($this->wheres) {
            $conditions = array_map(
                fn($w) => "{$w[0]} = '{$w[1]}'",
                $this->wheres
            );
            $sql .= ' WHERE ' . implode(' AND ', $conditions);
        }
        if ($this->limit) $sql .= ' LIMIT ' . $this->limit;
        return $sql;
    }
}

$query = (new QueryBuilder())
    ->table('users')
    ->select('id', 'name', 'email')
    ->whereStatus('active')
    ->whereRole('admin')
    ->limit(10);

echo $query;
?>
```


## Resources

- [Magic Methods](https://www.php.net/manual/en/language.oop5.magic.php) — Complete PHP magic methods documentation

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*