---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-di-basics"
---

# Dependency Injection Fundamentals

Dependency Injection (DI) is a design pattern where objects receive their dependencies from external sources rather than creating them internally.

## Without DI (Tight Coupling)

```php
<?php
// BAD: Creates its own dependencies
class UserService {
    private MySQLUserRepository $repository;
    private SmtpMailer $mailer;
    
    public function __construct() {
        $this->repository = new MySQLUserRepository();  // Hard-coded!
        $this->mailer = new SmtpMailer();               // Hard-coded!
    }
}

// Problems:
// - Can't swap implementations
// - Hard to test
// - Hidden dependencies
```

## With DI (Loose Coupling)

```php
<?php
// GOOD: Receives dependencies
class UserService {
    public function __construct(
        private UserRepositoryInterface $repository,  // Interface!
        private MailerInterface $mailer               // Interface!
    ) {}
}

// Benefits:
// - Easy to swap implementations
// - Easy to test (mock dependencies)
// - Explicit dependencies
```

## Types of Dependency Injection

### Constructor Injection (Preferred)

```php
<?php
class OrderService {
    public function __construct(
        private OrderRepository $orders,
        private PaymentGateway $payments,
        private Logger $logger
    ) {}
}
```

### Setter Injection

```php
<?php
class ReportGenerator {
    private ?Logger $logger = null;
    
    public function setLogger(Logger $logger): void {
        $this->logger = $logger;
    }
}
```

### Interface Injection

```php
<?php
interface LoggerAware {
    public function setLogger(Logger $logger): void;
}

class Service implements LoggerAware {
    private Logger $logger;
    
    public function setLogger(Logger $logger): void {
        $this->logger = $logger;
    }
}
```

## Manual Wiring

```php
<?php
// Composition root - wire everything together
$pdo = new PDO('mysql:host=localhost;dbname=app', 'user', 'pass');
$logger = new FileLogger('/var/log/app.log');
$mailer = new SmtpMailer('smtp.example.com');

$userRepository = new MySQLUserRepository($pdo);
$userService = new UserService($userRepository, $mailer);

$orderRepository = new MySQLOrderRepository($pdo);
$paymentGateway = new StripeGateway($_ENV['STRIPE_KEY']);
$orderService = new OrderService($orderRepository, $paymentGateway, $logger);
```

## Code Examples

**DI enabling testable code with mock implementations**

```php
<?php
declare(strict_types=1);

// DI enables easy testing
interface UserRepository {
    public function findByEmail(string $email): ?User;
    public function save(User $user): void;
}

interface PasswordHasher {
    public function hash(string $password): string;
    public function verify(string $password, string $hash): bool;
}

class RegistrationService {
    public function __construct(
        private UserRepository $users,
        private PasswordHasher $hasher
    ) {}
    
    public function register(string $email, string $password): User {
        if ($this->users->findByEmail($email)) {
            throw new DomainException('Email already exists');
        }
        
        $user = new User(
            email: $email,
            passwordHash: $this->hasher->hash($password)
        );
        
        $this->users->save($user);
        return $user;
    }
}

// In production:
$service = new RegistrationService(
    new MySQLUserRepository($pdo),
    new BcryptHasher()
);

// In tests:
class InMemoryUserRepository implements UserRepository {
    private array $users = [];
    
    public function findByEmail(string $email): ?User {
        foreach ($this->users as $user) {
            if ($user->email === $email) return $user;
        }
        return null;
    }
    
    public function save(User $user): void {
        $this->users[] = $user;
    }
}

class FakeHasher implements PasswordHasher {
    public function hash(string $password): string {
        return "hashed:$password";
    }
    
    public function verify(string $password, string $hash): bool {
        return $hash === "hashed:$password";
    }
}

// Test with fakes
$service = new RegistrationService(
    new InMemoryUserRepository(),
    new FakeHasher()
);
?>
```


## Resources

- [Dependency Injection](https://php-di.org/doc/understanding-di.html) — Understanding Dependency Injection in PHP

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*