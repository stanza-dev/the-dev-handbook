---
source_course: "php-databases"
source_lesson: "php-databases-active-record-basics"
---

# Active Record Implementation

## Introduction

The Active Record pattern wraps database rows in objects that handle their own persistence. Each object represents a row, knows how to save and load itself, and provides an intuitive API for CRUD operations. It trades strict separation of concerns for developer productivity.

## Key Concepts

- **Active Record**: An object that wraps a row in a database table, encapsulates database access, and adds domain logic on that data.
- **Fillable/Guarded**: Properties that control which attributes can be set via mass assignment, preventing security vulnerabilities.
- **Dirty Tracking**: Comparing current attribute values against original values to only update columns that actually changed.

## Real World Context

Laravel's Eloquent, Ruby on Rails' ActiveRecord, and CakePHP's ORM all implement this pattern. Understanding the underlying mechanics helps you debug ORM behavior, customize model operations, and recognize when the pattern's trade-offs work against you.

## Deep Dive

The base Active Record class provides CRUD operations:

```php
<?php
abstract class ActiveRecord {
    protected static string $table = '';
    protected static string $primaryKey = 'id';
    protected static array $fillable = [];
    
    protected array $attributes = [];
    protected array $original = [];
    protected bool $exists = false;
    protected static ?PDO $pdo = null;
    
    public static function setConnection(PDO $pdo): void {
        static::$pdo = $pdo;
    }
    
    public function __get(string $name): mixed {
        return $this->attributes[$name] ?? null;
    }
    
    public function __set(string $name, mixed $value): void {
        if (in_array($name, static::$fillable, true)) {
            $this->attributes[$name] = $value;
        }
    }
    
    public static function find(int $id): ?static {
        $table = static::$table;
        $pk = static::$primaryKey;
        
        // WARNING: $table and $pk are interpolated into SQL.
        // These MUST come from the class definition, never user input.
        // Column/table names cannot be parameterized with prepared statements.
        $stmt = static::$pdo->prepare(
            "SELECT * FROM {$table} WHERE {$pk} = :id"
        );
        $stmt->execute(['id' => $id]);
        $row = $stmt->fetch();
        
        return $row ? static::hydrate($row) : null;
    }
    
    protected static function hydrate(array $row): static {
        $instance = new static();
        $instance->attributes = $row;
        $instance->original = $row;
        $instance->exists = true;
        return $instance;
    }
    
    public function save(): bool {
        return $this->exists ? $this->update() : $this->insert();
    }
    
    protected function insert(): bool {
        $table = static::$table;
        $columns = array_keys($this->attributes);
        $placeholders = array_map(fn($c) => ":$c", $columns);
        
        $sql = sprintf(
            'INSERT INTO %s (%s) VALUES (%s)',
            $table,
            implode(', ', $columns),
            implode(', ', $placeholders)
        );
        
        $stmt = static::$pdo->prepare($sql);
        $result = $stmt->execute($this->attributes);
        
        if ($result) {
            $this->attributes[static::$primaryKey] = (int) static::$pdo->lastInsertId();
            $this->exists = true;
            $this->original = $this->attributes;
        }
        return $result;
    }
    
    protected function update(): bool {
        $dirty = $this->getDirty();
        if (empty($dirty)) {
            return true; // Nothing to update
        }
        
        $table = static::$table;
        $pk = static::$primaryKey;
        $sets = array_map(fn($c) => "$c = :$c", array_keys($dirty));
        
        $sql = sprintf(
            'UPDATE %s SET %s WHERE %s = :pk_value',
            $table,
            implode(', ', $sets),
            $pk
        );
        
        $params = $dirty;
        $params['pk_value'] = $this->attributes[$pk];
        
        $stmt = static::$pdo->prepare($sql);
        $result = $stmt->execute($params);
        
        if ($result) {
            $this->original = $this->attributes;
        }
        return $result;
    }
    
    protected function getDirty(): array {
        return array_diff_assoc($this->attributes, $this->original);
    }
    
    public function delete(): bool {
        $table = static::$table;
        $pk = static::$primaryKey;
        $stmt = static::$pdo->prepare(
            "DELETE FROM {$table} WHERE {$pk} = :id"
        );
        return $stmt->execute(['id' => $this->attributes[$pk]]);
    }
}
```

Critical security note: the `find()`, `insert()`, `update()`, and `delete()` methods interpolate `$table` and `$primaryKey` directly into SQL strings. These values come from `protected static` class properties, which is safe because they are set by the developer, not by user input. However, if you ever allow user input to influence table or column names, you introduce a SQL injection vulnerability. Column and table identifiers cannot be parameterized with prepared statements.

## Common Pitfalls

1. **Allowing user input to set table or column names** — Prepared statement parameters only work for values, not identifiers. Always validate identifiers against an allowlist if they come from any external source.
2. **Skipping the fillable guard** — Without `$fillable`, mass assignment can set sensitive columns like `is_admin` or `role` if the user includes them in the request payload.

## Best Practices

1. **Always define $fillable explicitly** — List only the columns users are allowed to set. Never use a "guarded = []" (empty guard) approach in production.
2. **Use dirty tracking to minimize UPDATE queries** — Only send changed columns to the database. This reduces query complexity and avoids triggering unnecessary triggers or audit logs.

## Summary

- Active Record objects represent database rows and handle their own CRUD operations.
- The `$fillable` array prevents mass assignment vulnerabilities by whitelisting settable attributes.
- Dirty tracking compares current values against originals to only update changed columns.
- Table and column names are interpolated into SQL — they must never come from user input.

## Code Examples

**Concrete Active Record model with password handling**

```php
<?php
declare(strict_types=1);

class User extends ActiveRecord {
    protected static string $table = 'users';
    protected static array $fillable = ['name', 'email', 'status'];
    
    public function setPassword(string $password): void {
        $this->attributes['password_hash'] = password_hash(
            $password, PASSWORD_DEFAULT
        );
    }
    
    public function verifyPassword(string $password): bool {
        return password_verify(
            $password, $this->attributes['password_hash'] ?? ''
        );
    }
}

// Usage
User::setConnection($pdo);
$user = new User();
$user->name = 'John';
$user->email = 'john@example.com';
$user->setPassword('secret123');
$user->save();
echo $user->id; // Auto-populated after insert
?>
```


## Resources

- [Active Record Pattern](https://www.martinfowler.com/eaaCatalog/activeRecord.html) — Martin Fowler's Active Record pattern description

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*