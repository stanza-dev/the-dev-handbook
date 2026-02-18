---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-magic-methods"
---

# Magic Methods

Magic methods are special methods that PHP calls automatically in certain situations. They all start with double underscores.

## Constructor & Destructor

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
        // Connection is automatically closed
    }
}
```

## Property Overloading

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
$obj->name = 'John';     // Calls __set
echo $obj->name;         // Calls __get
var_dump(isset($obj->name));  // Calls __isset
unset($obj->name);       // Calls __unset
```

## Method Overloading

```php
<?php
class ApiClient {
    public function __call(string $method, array $args): mixed {
        // Convert getUsers() to GET /users
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
$users = $client->getUsers();  // GET /users
$posts = ApiClient::getPosts(); // Static call
```

## String Conversion

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

## Cloning

```php
<?php
class Order {
    public function __construct(
        public int $id,
        public array $items,
        public DateTime $createdAt
    ) {}
    
    public function __clone(): void {
        // Deep clone the DateTime object
        $this->createdAt = clone $this->createdAt;
        $this->id = 0;  // Reset ID for the clone
    }
}

$order1 = new Order(1, ['item1'], new DateTime());
$order2 = clone $order1;
$order2->id;  // 0 (reset by __clone)
```

## Serialization

```php
<?php
class User {
    public function __construct(
        public string $name,
        public string $password  // Don't serialize this!
    ) {}
    
    public function __sleep(): array {
        // Only serialize these properties
        return ['name'];
    }
    
    public function __wakeup(): void {
        // Reinitialize after unserialize
        $this->password = '';
    }
}
```

## Code Examples

**Fluent query builder with magic methods**

```php
<?php
declare(strict_types=1);

// Fluent query builder using magic methods
class QueryBuilder {
    private string $table = '';
    private array $wheres = [];
    private array $selects = ['*'];
    private ?int $limit = null;
    
    public function __call(string $method, array $args): self {
        // where{Column}($value) -> where(column, value)
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
        
        if ($this->limit) {
            $sql .= ' LIMIT ' . $this->limit;
        }
        
        return $sql;
    }
}

$query = (new QueryBuilder())
    ->table('users')
    ->select('id', 'name', 'email')
    ->whereStatus('active')  // Magic method!
    ->whereRole('admin')     // Magic method!
    ->limit(10);

echo $query;
// SELECT id, name, email FROM users WHERE status = 'active' AND role = 'admin' LIMIT 10
?>
```


## Resources

- [Magic Methods](https://www.php.net/manual/en/language.oop5.magic.php) — Complete PHP magic methods documentation

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*