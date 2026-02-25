---
source_course: "php-modern-features"
source_lesson: "php-modern-features-enum-basics"
---

# Enum Fundamentals

## Introduction
Enumerations (enums), introduced in PHP 8.1, represent a fixed set of possible values. They replace the error-prone pattern of string or integer constants with a type-safe, first-class language construct.

## Key Concepts
- **Unit Enum**: A basic enum with named cases but no associated values.
- **Enum Case**: A single named value in an enumeration, e.g. `Status::Active`.
- **`cases()` Method**: A built-in static method that returns an array of all enum cases.

## Real World Context
Before enums, PHP developers used class constants (`const STATUS_ACTIVE = 'active'`) or plain strings. These had no type safety — any string could be passed where a status was expected. Enums fix this by making invalid values a type error, caught at both runtime and by static analyzers.

## Deep Dive

### Basic Enum (Unit Enum)

Declare an enum with the `enum` keyword:

```php
<?php
enum Status {
    case Pending;
    case Active;
    case Suspended;
    case Deleted;
}

$status = Status::Active;

if ($status === Status::Active) {
    echo "User is active";
}
```

Each case is a singleton instance. `Status::Active` is always the same object, making `===` comparison reliable.

### Why Enums Are Better Than Constants

Before enums, typos went unnoticed:

```php
<?php
// Old approach - error prone
const STATUS_ACTIVE = 'active';

function setStatus(string $status) {
    $user->status = $status;
}

setStatus('actve');  // Typo goes unnoticed!
```

With enums, the type system catches errors:

```php
<?php
enum Status { case Pending; case Active; }

function setStatus(Status $status) {
    // Only valid Status values allowed
}

setStatus(Status::Actve);  // Error! Invalid case
setStatus('active');        // Error! Wrong type
```

Both typos and wrong types are caught immediately.

### Enum Methods and Constants

Enums can have methods and constants:

```php
<?php
enum Status {
    case Pending;
    case Active;
    case Deleted;
    
    public const DEFAULT = self::Pending;
    
    public function label(): string {
        return match($this) {
            self::Pending => 'Waiting for Review',
            self::Active => 'Currently Active',
            self::Deleted => 'Removed',
        };
    }
    
    public static function activeStatuses(): array {
        return [self::Pending, self::Active];
    }
}

echo Status::Active->label();  // Currently Active
```

Methods on enums use `match($this)` to return different values for each case.

### Listing All Cases

The built-in `cases()` method returns all cases:

```php
<?php
enum Color {
    case Red;
    case Green;
    case Blue;
}

$colors = Color::cases();
// [Color::Red, Color::Green, Color::Blue]

foreach (Color::cases() as $color) {
    echo "<option value='{$color->name}'>{$color->name}</option>";
}
```

This is useful for generating form options, validation rules, and documentation.

### The `name` Property

Every enum case has a `name` property:

```php
<?php
$status = Status::Active;
echo $status->name;  // "Active" (string)
```

This returns the case name as a string, useful for serialization and display.

## Common Pitfalls
1. **Trying to instantiate enums with `new`** — Enums cannot be instantiated with `new Status()`. Use the case syntax `Status::Active` instead.
2. **Comparing with loose equality** — While `===` works correctly, `==` can produce unexpected results when comparing enums to strings or integers. Always use strict comparison.

## Best Practices
1. **Use enums for all fixed value sets** — Status codes, roles, categories, days of the week — anything with a fixed set of values should be an enum, not string constants.
2. **Add a `label()` method for display** — Keep display logic on the enum itself rather than scattering `match` statements across templates.

## Summary
- Enums represent fixed sets of values with full type safety.
- Unit enums have named cases but no associated scalar values.
- Every enum gets a `cases()` method and `name` property automatically.
- Enums can have methods, constants, and implement interfaces.
- Use enums instead of string/integer constants for type-safe code.

## Code Examples

**A Role enum with methods that encapsulate permission logic directly on each case**

```php
<?php
declare(strict_types=1);

enum Role {
    case Admin;
    case Editor;
    case Viewer;
    
    public function permissions(): array {
        return match($this) {
            self::Admin => ['read', 'write', 'delete', 'manage'],
            self::Editor => ['read', 'write'],
            self::Viewer => ['read'],
        };
    }
    
    public function canWrite(): bool {
        return in_array('write', $this->permissions());
    }
}

$role = Role::Editor;
echo $role->name;        // Editor
echo $role->canWrite();  // true
print_r($role->permissions());  // ['read', 'write']
?>
```


## Resources

- [Enumerations](https://www.php.net/manual/en/language.enumerations.php) — Official PHP enumeration documentation

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*