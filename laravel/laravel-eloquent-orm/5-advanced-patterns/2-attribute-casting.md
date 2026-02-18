---
source_course: "laravel-eloquent-orm"
source_lesson: "laravel-eloquent-orm-attribute-casting"
---

# Attribute Casting

Attribute casting automatically converts database values to common PHP data types. Instead of manually casting values everywhere, define casts once on your model.

## Basic Casting

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Get the attributes that should be cast.
     */
    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'is_admin' => 'boolean',
            'settings' => 'array',
            'birthday' => 'date',
            'balance' => 'decimal:2',
        ];
    }
}
```

## Available Cast Types

### Primitive Types

```php
protected function casts(): array
{
    return [
        'is_active' => 'boolean',      // true/false
        'age' => 'integer',             // int
        'price' => 'float',             // float
        'price' => 'double',            // same as float
        'amount' => 'decimal:2',        // string with 2 decimals
        'data' => 'string',             // string
    ];
}
```

### Date/Time Types

```php
protected function casts(): array
{
    return [
        'created_at' => 'datetime',           // Carbon instance
        'published_at' => 'datetime:Y-m-d',   // Custom format
        'birthday' => 'date',                 // Carbon (date only)
        'time_slot' => 'timestamp',           // Unix timestamp
        'expires_at' => 'immutable_datetime', // CarbonImmutable
    ];
}
```

Usage:

```php
$user = User::find(1);

$user->birthday;                    // Carbon instance
$user->birthday->age;               // 25
$user->birthday->format('F j, Y');  // "January 15, 1999"
$user->birthday->diffForHumans();   // "25 years ago"
```

### Array and JSON

```php
protected function casts(): array
{
    return [
        'settings' => 'array',             // JSON to PHP array
        'preferences' => 'json',           // Same as array
        'metadata' => 'object',            // JSON to stdClass
        'collection' => 'collection',      // JSON to Collection
        'options' => AsCollection::class,  // Same with class
    ];
}
```

Usage:

```php
// In database: {"theme":"dark","notifications":true}

$user = User::find(1);

// As array
$user->settings;                    // ['theme' => 'dark', 'notifications' => true]
$user->settings['theme'];           // 'dark'

// Update
$user->settings = ['theme' => 'light', 'notifications' => false];
$user->save();  // Saved as JSON string

// Merge values
$settings = $user->settings;
$settings['language'] = 'en';
$user->settings = $settings;
$user->save();
```

### Encrypted Casting

```php
protected function casts(): array
{
    return [
        'secret' => 'encrypted',              // Encrypted string
        'api_keys' => 'encrypted:array',      // Encrypted array
        'token' => 'encrypted:collection',    // Encrypted collection
        'credentials' => 'encrypted:object',  // Encrypted object
    ];
}
```

Data is encrypted at rest:

```php
$user->secret = 'my-api-key';  // Encrypted when saved
$user->secret;                  // Decrypted when accessed: "my-api-key"
```

### Hashed Casting (Laravel 10+)

```php
protected function casts(): array
{
    return [
        'password' => 'hashed',
    ];
}
```

```php
$user->password = 'secret';  // Automatically hashed!
// No need for Hash::make()
```

## Enum Casting

Cast to PHP enums:

```php
// app/Enums/UserStatus.php
enum UserStatus: string
{
    case Pending = 'pending';
    case Active = 'active';
    case Suspended = 'suspended';
}
```

```php
// In User model
protected function casts(): array
{
    return [
        'status' => UserStatus::class,
    ];
}
```

Usage:

```php
$user = User::find(1);

$user->status;                           // UserStatus::Active
$user->status === UserStatus::Active;    // true
$user->status->value;                    // "active"
$user->status->name;                     // "Active"

// Set using enum
$user->status = UserStatus::Suspended;

// Query
User::where('status', UserStatus::Active)->get();
```

## Custom Cast Classes

For complex casting logic:

```bash
php artisan make:cast Json
```

```php
<?php

namespace App\Casts;

use Illuminate\Contracts\Database\Eloquent\CastsAttributes;
use Illuminate\Database\Eloquent\Model;

class Money implements CastsAttributes
{
    public function __construct(
        protected string $currency = 'USD'
    ) {}

    /**
     * Cast the given value (from database to PHP).
     */
    public function get(Model $model, string $key, mixed $value, array $attributes): Money
    {
        return new Money(
            $value,
            $this->currency
        );
    }

    /**
     * Prepare the given value for storage (from PHP to database).
     */
    public function set(Model $model, string $key, mixed $value, array $attributes): int
    {
        return $value instanceof Money
            ? $value->cents
            : $value;
    }
}
```

Usage:

```php
protected function casts(): array
{
    return [
        'price' => Money::class . ':USD',
        'cost' => Money::class . ':EUR',
    ];
}
```

## Castable Classes

Value objects that handle their own casting:

```php
use Illuminate\Contracts\Database\Eloquent\Castable;

class Address implements Castable
{
    public static function castUsing(array $arguments): string
    {
        return AddressCast::class;
    }
}
```

```php
protected function casts(): array
{
    return [
        'address' => Address::class,
    ];
}

// Usage
$user->address = new Address('123 Main St', 'New York', 'NY');
$user->address->city;  // "New York"
```

## Resources

- [Attribute Casting](https://laravel.com/docs/12.x/eloquent-mutators#attribute-casting) — Official documentation on attribute casting

---

> 📘 *This lesson is part of the [Laravel Eloquent ORM](https://stanza.dev/courses/laravel-eloquent-orm) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*