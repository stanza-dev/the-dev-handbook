---
source_course: "laravel-eloquent-orm"
source_lesson: "laravel-eloquent-orm-accessors-mutators"
---

# Accessors and Mutators

Accessors and mutators allow you to transform Eloquent attribute values when retrieving or setting them on model instances. They're perfect for formatting data consistently.

## Defining Accessors

Accessors transform attributes when you **get** them:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Get the user's full name.
     * Combines first_name and last_name.
     */
    protected function fullName(): Attribute
    {
        return Attribute::make(
            get: fn () => "{$this->first_name} {$this->last_name}",
        );
    }

    /**
     * Always return email in lowercase.
     */
    protected function email(): Attribute
    {
        return Attribute::make(
            get: fn (string $value) => strtolower($value),
        );
    }

    /**
     * Format the created_at for display.
     */
    protected function createdAtFormatted(): Attribute
    {
        return Attribute::make(
            get: fn () => $this->created_at->format('F j, Y'),
        );
    }
}
```

### Using Accessors

```php
$user = User::find(1);

// first_name: "John", last_name: "Doe"
echo $user->full_name;  // "John Doe"

echo $user->email;      // "john@example.com" (lowercase)

echo $user->created_at_formatted;  // "January 15, 2024"
```

## Defining Mutators

Mutators transform attributes when you **set** them:

```php
class User extends Model
{
    /**
     * Always store email in lowercase.
     */
    protected function email(): Attribute
    {
        return Attribute::make(
            set: fn (string $value) => strtolower($value),
        );
    }

    /**
     * Hash password when setting.
     */
    protected function password(): Attribute
    {
        return Attribute::make(
            set: fn (string $value) => bcrypt($value),
        );
    }

    /**
     * Set multiple attributes from one input.
     */
    protected function fullName(): Attribute
    {
        return Attribute::make(
            set: function (string $value) {
                $parts = explode(' ', $value, 2);
                return [
                    'first_name' => $parts[0],
                    'last_name' => $parts[1] ?? '',
                ];
            },
        );
    }
}
```

### Using Mutators

```php
$user = new User;

$user->email = 'JOHN@EXAMPLE.COM';
// Stored as: "john@example.com"

$user->password = 'secret';
// Stored as: hashed value

$user->full_name = 'John Doe';
// Sets first_name = "John", last_name = "Doe"
```

## Combined Accessor and Mutator

```php
protected function phoneNumber(): Attribute
{
    return Attribute::make(
        // Get: Format for display
        get: fn (string $value) => sprintf(
            '(%s) %s-%s',
            substr($value, 0, 3),
            substr($value, 3, 3),
            substr($value, 6)
        ),
        // Set: Store only digits
        set: fn (string $value) => preg_replace('/[^0-9]/', '', $value),
    );
}
```

```php
$user->phone_number = '(555) 123-4567';
// Stored as: "5551234567"

echo $user->phone_number;
// Displayed as: "(555) 123-4567"
```

## Caching Accessors

For expensive computations:

```php
protected function avatarUrl(): Attribute
{
    return Attribute::make(
        get: fn () => $this->calculateGravatarUrl(),
    )->shouldCache();  // Cache the result
}

// The calculation only runs once per request
echo $user->avatar_url;  // Calculates
echo $user->avatar_url;  // Uses cached value
```

## Appending Accessors to JSON

```php
class User extends Model
{
    /**
     * Accessors to append to model's array/JSON.
     */
    protected $appends = ['full_name', 'avatar_url'];

    protected function fullName(): Attribute
    {
        return Attribute::make(
            get: fn () => "{$this->first_name} {$this->last_name}",
        );
    }
}
```

```php
$user->toArray();
// ['id' => 1, 'first_name' => 'John', ..., 'full_name' => 'John Doe']

$user->toJson();
// {"id": 1, "first_name": "John", ..., "full_name": "John Doe"}
```

## Real-World Examples

```php
class Product extends Model
{
    /**
     * Format price as currency.
     */
    protected function priceFormatted(): Attribute
    {
        return Attribute::make(
            get: fn () => '$' . number_format($this->price / 100, 2),
        );
    }

    /**
     * Store price in cents.
     */
    protected function price(): Attribute
    {
        return Attribute::make(
            get: fn (int $value) => $value / 100,  // Cents to dollars
            set: fn (float $value) => $value * 100, // Dollars to cents
        );
    }
}

class Post extends Model
{
    /**
     * Generate excerpt from body.
     */
    protected function excerpt(): Attribute
    {
        return Attribute::make(
            get: fn () => Str::limit(strip_tags($this->body), 150),
        );
    }

    /**
     * Calculate reading time.
     */
    protected function readingTime(): Attribute
    {
        return Attribute::make(
            get: function () {
                $words = str_word_count(strip_tags($this->body));
                $minutes = ceil($words / 200);  // 200 words per minute
                return "{$minutes} min read";
            },
        );
    }
}
```

## Resources

- [Accessors & Mutators](https://laravel.com/docs/12.x/eloquent-mutators#accessors-and-mutators) — Official documentation on accessors and mutators

---

> 📘 *This lesson is part of the [Laravel Eloquent ORM](https://stanza.dev/courses/laravel-eloquent-orm) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*