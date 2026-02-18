---
source_course: "php-modern-features"
source_lesson: "php-modern-features-enum-basics"
---

# Enum Fundamentals (PHP 8.1+)

Enumerations (enums) represent a fixed set of possible values. They're perfect for status codes, card suits, days of the week, etc.

## Basic Enum (Unit Enum)

```php
<?php
enum Status {
    case Pending;
    case Active;
    case Suspended;
    case Deleted;
}

// Usage
$status = Status::Active;

if ($status === Status::Active) {
    echo "User is active";
}
```

## Why Use Enums?

Before enums:
```php
<?php
// Old approach - error prone!
const STATUS_PENDING = 'pending';
const STATUS_ACTIVE = 'active';

function setStatus(string $status) {
    // Anyone can pass any string!
    $user->status = $status;
}

setStatus('actve');  // Typo goes unnoticed!
```

With enums:
```php
<?php
enum Status {
    case Pending;
    case Active;
}

function setStatus(Status $status) {
    // Type-safe - only valid Status values allowed
}

setStatus(Status::Actve);  // Error! Invalid case
setStatus('active');        // Error! Wrong type
```

## Enum Methods and Constants

```php
<?php
enum Status {
    case Pending;
    case Active;
    case Deleted;
    
    // Constant
    public const DEFAULT = self::Pending;
    
    // Method
    public function label(): string {
        return match($this) {
            self::Pending => 'Waiting for Review',
            self::Active => 'Currently Active',
            self::Deleted => 'Removed',
        };
    }
    
    // Static method
    public static function activeStatuses(): array {
        return [self::Pending, self::Active];
    }
}

echo Status::Active->label();  // Currently Active
```

## Listing All Cases

```php
<?php
enum Color {
    case Red;
    case Green;
    case Blue;
}

// Get all cases
$colors = Color::cases();
// [Color::Red, Color::Green, Color::Blue]

// Useful for forms
foreach (Color::cases() as $color) {
    echo "<option value='{$color->name}'>{$color->name}</option>";
}
```

## The `name` Property

```php
<?php
enum Status {
    case Active;
}

$status = Status::Active;
echo $status->name;  // "Active" (string)
```

## Resources

- [Enumerations](https://www.php.net/manual/en/language.enumerations.php) — Official PHP enumeration documentation

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*