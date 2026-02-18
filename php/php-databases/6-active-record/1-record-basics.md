---
source_course: "php-databases"
source_lesson: "php-databases-active-record-basics"
---

# Active Record Implementation

Active Record wraps database rows in objects that handle their own persistence.

## Base Active Record Class

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
    
    public function __isset(string $name): bool {
        return isset($this->attributes[$name]);
    }
    
    public static function find(int $id): ?static {
        $table = static::$table;
        $pk = static::$primaryKey;
        
        $stmt = static::$pdo->prepare("SELECT * FROM $table WHERE $pk = :id");
        $stmt->execute(['id' => $id]);
        $row = $stmt->fetch();
        
        if (!$row) {
            return null;
        }
        
        return static::hydrate($row);
    }
    
    protected static function hydrate(array $row): static {
        $instance = new static();
        $instance->attributes = $row;
        $instance->original = $row;
        $instance->exists = true;
        return $instance;
    }
    
    public function save(): bool {
        if ($this->exists) {
            return $this->update();
        }
        return $this->insert();
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
            return true;
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
        
        $stmt = static::$pdo->prepare("DELETE FROM $table WHERE $pk = :id");
        return $stmt->execute(['id' => $this->attributes[$pk]]);
    }
}
```

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*