---
source_course: "php-databases"
source_lesson: "php-databases-active-record-queries"
---

# Query Methods & Relationships

## Introduction

An Active Record class needs more than basic CRUD. Query methods like `where()`, `all()`, and `create()` provide a fluent API for common operations, while relationship methods let objects navigate to their associated records.

## Key Concepts

- **Static Query Methods**: Class-level methods like `User::where()` that return collections of model instances.
- **Relationships**: Methods on a model instance that load associated records (e.g., `$user->orders()` returns Order instances).
- **Mass Assignment**: Setting multiple attributes at once from an array, guarded by `$fillable`.

## Real World Context

In a typical controller, you might write `$user = User::find($id)` to load a user, `$user->orders()` to get their orders, and `User::where('status', 'active')->get()` to list active users. These patterns make controllers concise and readable.

## Deep Dive

Adding query methods to the base class:

```php
<?php
abstract class ActiveRecord {
    // ... previous code ...
    
    public static function all(): array {
        $table = static::$table;
        $stmt = static::$pdo->query("SELECT * FROM {$table}");
        return array_map([static::class, 'hydrate'], $stmt->fetchAll());
    }
    
    public static function where(
        string $column,
        mixed $value,
        string $operator = '='
    ): QueryBuilder {
        return (new QueryBuilder(static::$pdo, static::class))
            ->table(static::$table)
            ->where($column, $value, $operator);
    }
    
    public static function create(array $attributes): static {
        $instance = new static();
        foreach ($attributes as $key => $value) {
            $instance->$key = $value; // Goes through __set, checks $fillable
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

Concrete models define relationships as methods:

```php
<?php
class User extends ActiveRecord {
    protected static string $table = 'users';
    protected static array $fillable = ['name', 'email', 'password_hash'];
    
    public function orders(): array {
        return Order::where('user_id', $this->id)->get();
    }
    
    public function getDisplayName(): string {
        return ucfirst($this->name);
    }
}

class Order extends ActiveRecord {
    protected static string $table = 'orders';
    protected static array $fillable = ['user_id', 'status', 'total'];
    
    public function user(): ?User {
        return User::find($this->user_id);
    }
    
    public function items(): array {
        return OrderItem::where('order_id', $this->id)->get();
    }
}
```

Usage is clean and expressive:

```php
<?php
ActiveRecord::setConnection($pdo);

// Create
$user = User::create([
    'name' => 'John Doe',
    'email' => 'john@example.com',
]);

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
}
```

In PHP 8.5, the `#[\NoDiscard]` attribute can mark methods whose return values should not be ignored:

```php
<?php
class User extends ActiveRecord {
    #[\NoDiscard]
    public function save(): bool {
        return $this->exists ? $this->update() : $this->insert();
    }
}

// PHP 8.5 will warn if you ignore the return value:
$user->save();           // Warning: return value of save() is not used
$success = $user->save(); // OK
if (!$user->save()) {     // OK
    // handle error
}
```

This is especially useful for methods that return success/failure booleans.

## Common Pitfalls

1. **Calling relationship methods in a loop** — `$user->orders()` inside a foreach over users creates an N+1 problem. Use eager loading for batch access.
2. **Ignoring save() return values** — `save()` returns false on failure. Ignoring it means silent data loss. PHP 8.5's `#[\NoDiscard]` attribute helps catch this.

## Best Practices

1. **Add the `#[\NoDiscard]` attribute to save() and delete()** — PHP 8.5's attribute warns developers when they ignore the success/failure return value.
2. **Combine Active Record with eager loading** — For list pages, use query builder methods with JOINs or IN clauses instead of calling relationship methods in loops.

## Summary

- Static query methods (`find`, `where`, `create`, `all`) provide a fluent API for model operations.
- Relationship methods navigate between models but risk N+1 if called in loops.
- PHP 8.5's `#[\NoDiscard]` attribute warns when critical return values like `save()` are ignored.
- Always guard mass assignment with `$fillable` and avoid relationship methods inside loops.

## Code Examples

**Active Record with validation, hooks, and NoDiscard**

```php
<?php
declare(strict_types=1);

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
    }
    
    #[\NoDiscard]
    public function save(): bool {
        if (!$this->validate()) { return false; }
        $this->beforeCreate();
        return parent::save();
    }
    
    public function toArray(): array {
        return array_diff_key($this->attributes, array_flip(static::$hidden));
    }
}
?>
```


## Resources

- [PHP 8.5 NoDiscard Attribute](https://wiki.php.net/rfc/marking_return_value_as_important) — RFC for the NoDiscard attribute in PHP 8.5

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*