---
source_course: "php-databases"
source_lesson: "php-databases-active-record-queries"
---

# Query Methods & Relationships

Extend Active Record with fluent query methods and relationship support.

## Query Methods

```php
<?php
abstract class ActiveRecord {
    // ... previous code ...
    
    public static function all(): array {
        $table = static::$table;
        $stmt = static::$pdo->query("SELECT * FROM $table");
        return array_map([static::class, 'hydrate'], $stmt->fetchAll());
    }
    
    public static function where(string $column, mixed $value, string $operator = '='): QueryBuilder {
        return (new QueryBuilder(static::$pdo, static::class))
            ->table(static::$table)
            ->where($column, $value, $operator);
    }
    
    public static function create(array $attributes): static {
        $instance = new static();
        foreach ($attributes as $key => $value) {
            $instance->$key = $value;
        }
        $instance->save();
        return $instance;
    }
    
    public function fill(array $attributes): static {
        foreach ($attributes as $key => $value) {
            if (in_array($key, static::$fillable, true)) {
                $this->attributes[$key] = $value;
            }
        }
        return $this;
    }
    
    public function toArray(): array {
        return $this->attributes;
    }
}
```

## Concrete Model

```php
<?php
class User extends ActiveRecord {
    protected static string $table = 'users';
    protected static array $fillable = ['name', 'email', 'password_hash'];
    
    // Relationship: User has many Orders
    public function orders(): array {
        return Order::where('user_id', $this->id)->get();
    }
    
    // Accessor
    public function getDisplayName(): string {
        return ucfirst($this->name);
    }
    
    // Mutator (called before save)
    public function setPassword(string $password): void {
        $this->attributes['password_hash'] = password_hash($password, PASSWORD_DEFAULT);
    }
}

class Order extends ActiveRecord {
    protected static string $table = 'orders';
    protected static array $fillable = ['user_id', 'status', 'total'];
    
    // Relationship: Order belongs to User
    public function user(): ?User {
        return User::find($this->user_id);
    }
    
    // Relationship: Order has many Items
    public function items(): array {
        return OrderItem::where('order_id', $this->id)->get();
    }
}
```

## Usage Examples

```php
<?php
// Set connection
ActiveRecord::setConnection($pdo);

// Create
$user = User::create([
    'name' => 'John Doe',
    'email' => 'john@example.com',
]);
$user->setPassword('secret123');
$user->save();

// Read
$user = User::find(1);
echo $user->name;

// Query
$activeUsers = User::where('status', 'active')->get();
$recentOrders = Order::where('created_at', date('Y-m-d'), '>=')->get();

// Update
$user->name = 'Jane Doe';
$user->save();

// Delete
$user->delete();

// Relationships
$user = User::find(1);
foreach ($user->orders() as $order) {
    echo "Order #{$order->id}: \${$order->total}\n";
    foreach ($order->items() as $item) {
        echo "  - {$item->product_name}\n";
    }
}
```

## Code Examples

**Active Record with validation and lifecycle hooks**

```php
<?php
declare(strict_types=1);

// Complete Active Record with hooks and validation
abstract class ActiveRecord {
    protected static string $table = '';
    protected static string $primaryKey = 'id';
    protected static array $fillable = [];
    protected static array $hidden = ['password_hash'];
    
    protected array $attributes = [];
    protected array $original = [];
    protected bool $exists = false;
    protected array $errors = [];
    
    protected static ?PDO $pdo = null;
    
    public static function setConnection(PDO $pdo): void {
        static::$pdo = $pdo;
    }
    
    // Hooks - override in subclasses
    protected function beforeSave(): void {}
    protected function afterSave(): void {}
    protected function beforeCreate(): void {}
    protected function afterCreate(): void {}
    protected function beforeUpdate(): void {}
    protected function afterUpdate(): void {}
    protected function beforeDelete(): void {}
    protected function afterDelete(): void {}
    protected function validate(): bool { return true; }
    
    public function save(): bool {
        $this->beforeSave();
        
        if (!$this->validate()) {
            return false;
        }
        
        if ($this->exists) {
            $this->beforeUpdate();
            $result = $this->performUpdate();
            if ($result) $this->afterUpdate();
        } else {
            $this->beforeCreate();
            $result = $this->performInsert();
            if ($result) $this->afterCreate();
        }
        
        if ($result) $this->afterSave();
        return $result;
    }
    
    public function getErrors(): array {
        return $this->errors;
    }
    
    public function toArray(): array {
        return array_diff_key($this->attributes, array_flip(static::$hidden));
    }
    
    public function toJson(): string {
        return json_encode($this->toArray());
    }
}

// Example model with validation and hooks
class User extends ActiveRecord {
    protected static string $table = 'users';
    protected static array $fillable = ['name', 'email', 'status'];
    protected static array $hidden = ['password_hash'];
    
    protected function validate(): bool {
        $this->errors = [];
        
        if (empty($this->attributes['name'])) {
            $this->errors['name'] = 'Name is required';
        }
        
        if (empty($this->attributes['email'])) {
            $this->errors['email'] = 'Email is required';
        } elseif (!filter_var($this->attributes['email'], FILTER_VALIDATE_EMAIL)) {
            $this->errors['email'] = 'Invalid email format';
        }
        
        return empty($this->errors);
    }
    
    protected function beforeCreate(): void {
        $this->attributes['created_at'] = date('Y-m-d H:i:s');
        $this->attributes['status'] = $this->attributes['status'] ?? 'active';
    }
    
    protected function beforeUpdate(): void {
        $this->attributes['updated_at'] = date('Y-m-d H:i:s');
    }
    
    public function setPassword(string $password): void {
        $this->attributes['password_hash'] = password_hash($password, PASSWORD_DEFAULT);
    }
    
    public function verifyPassword(string $password): bool {
        return password_verify($password, $this->attributes['password_hash'] ?? '');
    }
}

// Usage
$user = new User();
$user->name = 'John';
$user->email = 'invalid';

if (!$user->save()) {
    print_r($user->getErrors());
    // ['email' => 'Invalid email format']
}

$user->email = 'john@example.com';
$user->setPassword('secret123');
$user->save();  // Success!

echo $user->toJson();
// {"id":1,"name":"John","email":"john@example.com","status":"active"}
// Note: password_hash is hidden
?>
```


---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*