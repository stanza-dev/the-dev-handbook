---
source_course: "php-modern-features"
source_lesson: "php-modern-features-property-hooks"
---

# Property Hooks

## Introduction
Property hooks, introduced in PHP 8.4, let you define custom get and set behavior directly on properties. They replace the traditional pattern of private properties with explicit getter/setter methods, similar to C# or Kotlin property accessors.

## Key Concepts
- **Property Hook**: A `get` or `set` block attached to a property declaration that runs when the property is read or written.
- **Virtual Property**: A property with only a `get` hook and no backing storage — its value is computed on every access.
- **Asymmetric Visibility**: Different access levels for reading and writing a property, e.g. `public private(set)` (PHP 8.4+).

## Real World Context
In every PHP framework, you see entities with dozens of `getX()` and `setX()` methods that do little more than return or validate a value. Property hooks move that logic into the property declaration itself, making classes shorter and more declarative.

## Deep Dive

### Basic Syntax

Attach `get` and `set` hooks directly to a property:

```php
<?php
class User {
    public string $name {
        get => strtoupper($this->name);
        set => trim($value);
    }
}

$user = new User();
$user->name = '  john doe  ';  // Calls set hook, stores 'john doe'
echo $user->name;               // Calls get hook, outputs 'JOHN DOE'
```

The `set` hook receives the incoming value as `$value` and the result is stored. The `get` hook transforms the stored value on read.

### Virtual Properties

A property with only a `get` hook has no backing storage — it is computed every time:

```php
<?php
class Rectangle {
    public function __construct(
        public float $width,
        public float $height
    ) {}
    
    public float $area {
        get => $this->width * $this->height;
    }
    
    public float $perimeter {
        get => 2 * ($this->width + $this->height);
    }
}

$rect = new Rectangle(10, 5);
echo $rect->area;      // 50
echo $rect->perimeter; // 30
```

Virtual properties look and feel like regular properties to the caller, but are computed on access.

### Asymmetric Visibility (PHP 8.4)

Control read and write access independently:

```php
<?php
class BankAccount {
    public private(set) float $balance = 0;
    
    public function deposit(float $amount): void {
        $this->balance += $amount;  // OK - internal write
    }
}

$account = new BankAccount();
echo $account->balance;    // OK - public read
$account->balance = 100;   // Error! Private set
$account->deposit(100);    // OK - internal modification
```

The `public private(set)` syntax means anyone can read the property, but only the class itself can write to it.

### Validation in Set Hooks

Set hooks are ideal for validation:

```php
<?php
class Product {
    public float $price {
        set {
            if ($value < 0) {
                throw new InvalidArgumentException('Price cannot be negative');
            }
            $this->price = $value;
        }
    }
    
    public int $quantity {
        set(int $value) {
            $this->quantity = max(0, $value);
        }
    }
}
```

Validation happens transparently whenever the property is assigned, keeping invariants enforced.

### Temperature Converter Example

Property hooks enable elegant bidirectional computed properties:

```php
<?php
class Temperature {
    private float $celsius;
    
    public float $fahrenheit {
        get => $this->celsius * 9/5 + 32;
        set {
            $this->celsius = ($value - 32) * 5/9;
        }
    }
    
    public function __construct(float $celsius) {
        $this->celsius = $celsius;
    }
}

$temp = new Temperature(0);
echo $temp->fahrenheit;     // 32
$temp->fahrenheit = 212;    // Sets celsius to 100
```

Reading `fahrenheit` converts from celsius; writing `fahrenheit` converts back. The backing store is always celsius.

### Interface Properties with Hooks

Interfaces can declare property signatures:

```php
<?php
interface HasFullName {
    public string $fullName { get; }
}

class Person implements HasFullName {
    public function __construct(
        public string $firstName,
        public string $lastName
    ) {}
    
    public string $fullName {
        get => "$this->firstName $this->lastName";
    }
}
```

This lets interfaces require computed properties without dictating the implementation.

## Common Pitfalls
1. **Infinite recursion in get hooks** — Accessing `$this->name` inside the `get` hook of `$name` triggers the hook again. Use a backing property or the implicit storage to avoid loops.
2. **Forgetting explicit assignment in set hooks** — When using the block `set { ... }` syntax (not the arrow syntax), you must explicitly assign `$this->property = $value;` or the value is not stored.

## Best Practices
1. **Use virtual properties for computed values** — Replace `getArea()` methods with virtual properties for a cleaner API.
2. **Use asymmetric visibility instead of getters** — `public private(set)` is more idiomatic than a private property with a public getter method.

## Summary
- Property hooks attach `get` and `set` logic directly to property declarations.
- Virtual properties have only a `get` hook and no backing storage.
- Asymmetric visibility (`public private(set)`) controls read/write access independently.
- Set hooks are ideal for validation and normalization.
- Interface properties can require hooks, enabling contract-driven computed properties.

## Code Examples

**Entity using PHP 8.4 property hooks for validation, computed properties, and asymmetric visibility**

```php
<?php
declare(strict_types=1);

class Order {
    private array $items = [];
    
    public private(set) string $status = 'pending';
    
    public string $customerEmail {
        set {
            if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
                throw new InvalidArgumentException('Invalid email');
            }
            $this->customerEmail = strtolower($value);
        }
    }
    
    public float $total {
        get => array_sum(array_map(
            fn($item) => $item['price'] * $item['quantity'],
            $this->items
        ));
    }
    
    public string $formattedTotal {
        get => '$' . number_format($this->total, 2);
    }
    
    public function addItem(string $name, float $price, int $quantity): void {
        $this->items[] = compact('name', 'price', 'quantity');
    }
    
    public function markShipped(): void {
        $this->status = 'shipped';
    }
}

$order = new Order();
$order->customerEmail = 'JOHN@EXAMPLE.COM';
$order->addItem('Widget', 9.99, 2);
$order->addItem('Gadget', 19.99, 1);

echo $order->formattedTotal;  // $39.97
echo $order->status;          // pending
?>
```


## Resources

- [Property Hooks RFC](https://wiki.php.net/rfc/property-hooks) — RFC documentation for property hooks

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*