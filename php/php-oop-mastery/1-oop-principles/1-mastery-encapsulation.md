---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-encapsulation"
---

# Encapsulation & Information Hiding

Encapsulation bundles data with methods that operate on that data and restricts direct access to internal state.

## Visibility Modifiers

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

## Why Encapsulation Matters

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

## Asymmetric Visibility (PHP 8.4)

```php
<?php
class User {
    // Public read, private write
    public private(set) string $id;
    
    // Public read, protected write
    public protected(set) string $name;
    
    public function __construct(string $id, string $name) {
        $this->id = $id;
        $this->name = $name;
    }
}

$user = new User('123', 'John');
echo $user->id;      // OK - public read
$user->id = '456';   // Error! private(set)
```

## Getters and Setters vs Property Hooks

```php
<?php
// Traditional getters/setters
class TraditionalUser {
    private string $email;
    
    public function getEmail(): string {
        return $this->email;
    }
    
    public function setEmail(string $email): void {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException('Invalid email');
        }
        $this->email = strtolower($email);
    }
}

// PHP 8.4 Property Hooks
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

## Code Examples

**Well-encapsulated shopping cart class**

```php
<?php
declare(strict_types=1);

// Well-encapsulated shopping cart
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
    
    // Return a copy to prevent external modification
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