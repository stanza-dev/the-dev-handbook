---
source_course: "php-modern-features"
source_lesson: "php-modern-features-property-hooks"
---

# Property Hooks (PHP 8.4)

Property hooks allow you to define custom get and set behavior directly on properties, similar to getters/setters in other languages.

## Basic Syntax

```php
<?php
class User {
    public string $name {
        get => strtoupper($this->name);
        set => trim($value);
    }
}

$user = new User();
$user->name = '  john doe  ';  // Calls set hook
echo $user->name;               // Calls get hook: "JOHN DOE"
```

## Virtual Properties

Properties without backing storage:

```php
<?php
class Rectangle {
    public function __construct(
        public float $width,
        public float $height
    ) {}
    
    // Virtual property - computed on access
    public float $area {
        get => $this->width * $this->height;
    }
    
    // Read-only virtual property
    public float $perimeter {
        get => 2 * ($this->width + $this->height);
    }
}

$rect = new Rectangle(10, 5);
echo $rect->area;      // 50
echo $rect->perimeter; // 30
```

## Asymmetric Visibility (PHP 8.4)

Different visibility for get and set:

```php
<?php
class BankAccount {
    // Public read, private write
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

## Validation in Set Hooks

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
            $this->quantity = max(0, $value);  // Ensure non-negative
        }
    }
}
```

## Full Property Hook Syntax

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

## Interface Properties with Hooks

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

## Code Examples

**Entity using PHP 8.4 property hooks**

```php
<?php
declare(strict_types=1);

// Entity with property hooks for validation and computed properties
class Order {
    private array $items = [];
    
    // Asymmetric visibility
    public private(set) string $status = 'pending';
    
    // Validated property
    public string $customerEmail {
        set {
            if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
                throw new InvalidArgumentException('Invalid email');
            }
            $this->customerEmail = strtolower($value);
        }
    }
    
    // Virtual computed property
    public float $total {
        get => array_sum(array_map(
            fn($item) => $item['price'] * $item['quantity'],
            $this->items
        ));
    }
    
    // Virtual property with formatting
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
$order->customerEmail = 'JOHN@EXAMPLE.COM';  // Stored as lowercase
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