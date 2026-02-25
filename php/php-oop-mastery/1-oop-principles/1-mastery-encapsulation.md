---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-encapsulation"
---

# Encapsulation & Information Hiding

## Introduction
Encapsulation is one of the four pillars of object-oriented programming. It bundles data with methods that operate on that data and restricts direct access to internal state. In PHP 8.5, encapsulation is more expressive than ever with asymmetric visibility for static properties and final properties via constructor promotion.

## Key Concepts
- **Visibility Modifiers**: `public`, `protected`, and `private` control who can access properties and methods.
- **Information Hiding**: Exposing only what is necessary and hiding internal implementation details.
- **Asymmetric Visibility**: PHP 8.4+ allows different read/write visibilities; PHP 8.5 extends this to static properties.
- **Property Hooks**: PHP 8.4+ lets you define get/set logic directly on properties.

## Real World Context
Imagine a banking application. If the `balance` property is public, any code can set it to a negative number. Encapsulation forces all balance changes through validated methods like `deposit()` and `withdraw()`, preventing invalid states.

## Deep Dive

### Visibility Modifiers

PHP provides three visibility levels for properties and methods.

```php
<?php
class BankAccount {
    private float $balance = 0;      // Only this class
    protected string $accountType;   // This class + children
    public string $accountNumber;    // Anyone

    public function __construct(string $number, string $type = 'checking') {
        $this->accountNumber = $number;
        $this->accountType = $type;
    }
}
```

The `private` modifier is the most restrictive. Even child classes cannot access private members.

### Why Encapsulation Matters

Without encapsulation, objects can enter invalid states.

```php
<?php
// BAD: Public properties allow invalid state
class BadAccount {
    public float $balance = 0;
}
$account = new BadAccount();
$account->balance = -1000;  // Invalid negative balance!

// GOOD: Controlled access through methods
class GoodAccount {
    private float $balance = 0;

    public function deposit(float $amount): void {
        if ($amount <= 0) {
            throw new InvalidArgumentException('Amount must be positive');
        }
        $this->balance += $amount;
    }

    public function withdraw(float $amount): void {
        if ($amount <= 0) {
            throw new InvalidArgumentException('Amount must be positive');
        }
        if ($amount > $this->balance) {
            throw new InsufficientFundsException('Insufficient funds');
        }
        $this->balance -= $amount;
    }

    public function getBalance(): float {
        return $this->balance;
    }
}
```

By making `$balance` private, you guarantee the account never has an invalid balance.

### Asymmetric Visibility (PHP 8.4+, extended in PHP 8.5)

PHP 8.4 introduced asymmetric visibility for instance properties. PHP 8.5 extends this to static properties as well.

```php
<?php
class User {
    public private(set) string $id;
    public protected(set) string $name;

    public function __construct(string $id, string $name) {
        $this->id = $id;
        $this->name = $name;
    }
}

$user = new User('123', 'John');
echo $user->id;      // OK - public read
// $user->id = '456'; // Error! private(set)

// PHP 8.5: Asymmetric visibility for static properties
class Config {
    public private(set) static string $environment = 'production';

    public static function setEnvironment(string $env): void {
        self::$environment = $env; // Allowed inside class
    }
}
echo Config::$environment;  // OK
// Config::$environment = 'dev'; // Error!
```

This eliminates the need for many getter methods while still protecting write access.

### Final Properties via Constructor Promotion (PHP 8.5)

PHP 8.5 allows marking promoted properties as `final`, preventing child classes from overriding them.

```php
<?php
class BaseEntity {
    public function __construct(
        public final readonly string $id,
    ) {}
}

class User extends BaseEntity {
    // Cannot redeclare $id - it is final
}
```

This provides an additional layer of encapsulation for inheritance hierarchies.

### Property Hooks (PHP 8.4+)

```php
<?php
class ModernUser {
    public string $email {
        get => $this->email;
        set {
            if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
                throw new InvalidArgumentException('Invalid email');
            }
            $this->email = strtolower($value);
        }
    }
}
```

Property hooks keep validation close to the data and reduce boilerplate.

## Common Pitfalls
1. **Making everything public for convenience** — This defeats the purpose of encapsulation. Start with `private` and only increase visibility when truly needed.
2. **Returning mutable references** — Returning arrays or objects by reference allows callers to modify internal state. Return copies or use `readonly`.

## Best Practices
1. **Default to private** — Only expose what is necessary. Use asymmetric visibility when read access is needed but write access should be restricted.
2. **Validate in setters or hooks** — Always validate data before storing it. Property hooks make this clean and concise.

## Summary
- Encapsulation protects internal state by controlling access through visibility modifiers.
- PHP 8.5 extends asymmetric visibility to static properties and supports final promoted properties.
- Property hooks (PHP 8.4+) replace traditional getter/setter boilerplate.
- Default to `private` and increase visibility only when needed.

## Code Examples

**Well-encapsulated shopping cart class**

```php
<?php
declare(strict_types=1);

class ShoppingCart {
    private array $items = [];

    public function addItem(string $productId, int $quantity, float $price): void {
        if ($quantity <= 0) {
            throw new InvalidArgumentException('Quantity must be positive');
        }
        if (isset($this->items[$productId])) {
            $this->items[$productId]['quantity'] += $quantity;
        } else {
            $this->items[$productId] = [
                'quantity' => $quantity,
                'price' => $price,
            ];
        }
    }

    public function removeItem(string $productId): void {
        unset($this->items[$productId]);
    }

    public function getTotal(): float {
        return array_reduce(
            $this->items,
            fn($total, $item) => $total + ($item['price'] * $item['quantity']),
            0.0
        );
    }

    public function getItemCount(): int {
        return array_sum(array_column($this->items, 'quantity'));
    }

    public function getItems(): array {
        return $this->items;
    }
}
?>
```


## Resources

- [Visibility](https://www.php.net/manual/en/language.oop5.visibility.php) — PHP visibility modifiers documentation

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*