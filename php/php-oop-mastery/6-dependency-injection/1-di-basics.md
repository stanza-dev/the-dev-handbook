---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-di-basics"
---

# Dependency Injection Fundamentals

## Introduction
Dependency Injection (DI) is a design pattern where objects receive their dependencies from external sources rather than creating them internally. It is the practical application of the Dependency Inversion Principle and is essential for writing testable, maintainable code.

## Key Concepts
- **Dependency**: An object that another object needs to function.
- **Injection**: Providing dependencies from outside rather than creating them inside.
- **Inversion of Control**: The caller controls what dependencies are provided, not the class itself.
- **Composition Root**: The single place in your application where all dependencies are wired together.

## Real World Context
Without DI, a `UserService` creates its own `MySQLRepository` and `SmtpMailer` internally. Testing requires a real database and mail server. With DI, you inject interfaces and can swap in `InMemoryRepository` and `FakeMailer` for tests.

## Deep Dive

### Without DI (Tight Coupling)

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
// - Cannot swap implementations
// - Hard to test
// - Hidden dependencies
```

The class hides its dependencies and tightly couples to specific implementations.

### With DI (Loose Coupling)

```php
<?php
// GOOD: Receives dependencies
class UserService {
    public function __construct(
        private UserRepositoryInterface $repository,
        private MailerInterface $mailer
    ) {}
}

// Benefits:
// - Easy to swap implementations
// - Easy to test (mock dependencies)
// - Explicit dependencies visible in constructor
```

Dependencies are explicit, injectable, and swappable.

### Types of Dependency Injection

**Constructor Injection (Preferred)**

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

Constructor injection makes all dependencies required and visible.

**Setter Injection (For Optional Dependencies)**

```php
<?php
class ReportGenerator {
    private ?Logger $logger = null;

    public function setLogger(Logger $logger): void {
        $this->logger = $logger;
    }
}
```

Use setter injection only for optional dependencies that have sensible defaults.

**Interface Injection**

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

Interface injection formalizes the setter pattern through an interface contract.

### Manual Wiring (Composition Root)

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

The composition root is the only place that knows about concrete classes.

## Common Pitfalls
1. **Injecting the container** — Passing the DI container itself as a dependency is the Service Locator anti-pattern. It hides real dependencies.
2. **Too many constructor parameters** — If a constructor has more than 4-5 parameters, the class likely has too many responsibilities.

## Best Practices
1. **Always use constructor injection for required dependencies** — It makes dependencies explicit and ensures objects are fully initialized.
2. **Depend on interfaces, not classes** — Inject `LoggerInterface` not `FileLogger` for maximum flexibility.

## Summary
- DI provides dependencies from outside rather than creating them internally.
- Constructor injection is preferred because it makes dependencies explicit and required.
- The composition root is the single place where all dependencies are wired.
- Depend on interfaces for loose coupling and testability.

## Code Examples

**DI enabling testable code with mock implementations**

```php
<?php
declare(strict_types=1);

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

// Production
$service = new RegistrationService(
    new MySQLUserRepository($pdo),
    new BcryptHasher()
);

// Tests
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