---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-di-testing"
---

# DI for Testing & Real-World Patterns

## Introduction
Dependency Injection's greatest payoff is testability. By injecting dependencies, you can replace real implementations with test doubles. This lesson covers testing patterns, common DI architectures, and real-world integration.

## Key Concepts
- **Test Doubles**: Fakes, stubs, mocks, and spies that replace real dependencies in tests.
- **In-Memory Implementations**: Lightweight implementations for testing without external services.
- **Provider Pattern**: A factory-like pattern for creating context-specific instances.
- **Autowiring**: Automatic resolution of dependencies based on type hints.

## Real World Context
Your team writes a payment service that calls an external API. Without DI, every test makes real HTTP requests, is slow, flaky, and costs money. With DI, you inject a `FakePaymentGateway` that returns predictable responses, making tests fast and reliable.

## Deep Dive

### Test Doubles with DI

```php
<?php
interface NotificationService {
    public function send(string $to, string $message): bool;
}

class SmtpNotificationService implements NotificationService {
    public function send(string $to, string $message): bool {
        // Real SMTP sending
        return mail($to, 'Notification', $message);
    }
}

// Fake for testing
class FakeNotificationService implements NotificationService {
    public array $sent = [];

    public function send(string $to, string $message): bool {
        $this->sent[] = ['to' => $to, 'message' => $message];
        return true;
    }

    public function assertSentTo(string $to): void {
        $found = array_filter($this->sent, fn($s) => $s['to'] === $to);
        if (empty($found)) {
            throw new AssertionError("No notification sent to $to");
        }
    }
}

class OrderService {
    public function __construct(
        private NotificationService $notifications
    ) {}

    public function placeOrder(string $customerEmail): void {
        // ... place order logic
        $this->notifications->send($customerEmail, 'Order confirmed!');
    }
}

// Test
$fakeNotifier = new FakeNotificationService();
$service = new OrderService($fakeNotifier);
$service->placeOrder('customer@example.com');
$fakeNotifier->assertSentTo('customer@example.com');
```

The fake captures all notifications, letting you assert exactly what was sent without sending real emails.

### Provider Pattern

```php
<?php
interface LoggerProvider {
    public function getLogger(string $channel): Logger;
}

class DefaultLoggerProvider implements LoggerProvider {
    public function getLogger(string $channel): Logger {
        return match($channel) {
            'error' => new FileLogger('/var/log/errors.log'),
            'audit' => new DatabaseLogger($this->pdo),
            default => new FileLogger('/var/log/app.log'),
        };
    }
}

class UserService {
    private Logger $logger;

    public function __construct(LoggerProvider $loggerProvider) {
        $this->logger = $loggerProvider->getLogger('audit');
    }
}
```

The Provider pattern is useful when you need different instances of the same interface based on context.

### Real-World DI Architecture

```php
<?php
// bootstrap.php - Composition Root
function createContainer(): Container {
    $container = new Container();

    // Infrastructure
    $container->singleton(PDO::class, function() {
        return new PDO($_ENV['DATABASE_URL']);
    });

    // Repositories
    $container->bind(UserRepositoryInterface::class, MySQLUserRepository::class);
    $container->bind(OrderRepositoryInterface::class, MySQLOrderRepository::class);

    // Services
    $container->bind(MailerInterface::class, SmtpMailer::class);
    $container->bind(PaymentGateway::class, StripeGateway::class);

    return $container;
}

// In request handler
$container = createContainer();
$controller = $container->make(UserController::class);
$response = $controller->register($request);
```

The composition root wires everything once. All other code depends only on interfaces.

### Testing the Full Stack

```php
<?php
class OrderServiceTest extends TestCase {
    private InMemoryOrderRepository $orders;
    private FakePaymentGateway $payments;
    private FakeNotificationService $notifications;
    private OrderService $service;

    protected function setUp(): void {
        $this->orders = new InMemoryOrderRepository();
        $this->payments = new FakePaymentGateway();
        $this->notifications = new FakeNotificationService();

        $this->service = new OrderService(
            $this->orders,
            $this->payments,
            $this->notifications
        );
    }

    public function testPlaceOrderChargesPayment(): void {
        $this->service->placeOrder('user@test.com', 99.99);

        $this->assertTrue($this->payments->wasCharged(99.99));
        $this->notifications->assertSentTo('user@test.com');
        $this->assertCount(1, $this->orders->findAll());
    }
}
```

Every external dependency is replaced with a test double. Tests are fast, deterministic, and isolated.

## Common Pitfalls
1. **Testing implementation details** — Test behavior and outcomes, not that specific methods were called in a specific order.
2. **Over-mocking** — If you mock everything, your tests verify nothing. Use fakes with real logic for important collaborators.

## Best Practices
1. **Create In-Memory implementations** — For each repository interface, create an array-backed implementation for tests.
2. **Keep the composition root simple** — It should only wire dependencies, not contain business logic.

## Summary
- DI enables replacing real dependencies with test doubles for fast, reliable tests.
- Fakes with assertion methods are more useful than simple stubs.
- The Provider pattern handles context-dependent dependency creation.
- The composition root is the single wiring point; all other code uses interfaces.

## Code Examples

**In-memory cache for testing with DI**

```php
<?php
declare(strict_types=1);

interface Cache {
    public function get(string $key): mixed;
    public function set(string $key, mixed $value, int $ttl = 3600): void;
    public function has(string $key): bool;
}

class InMemoryCache implements Cache {
    private array $store = [];

    public function get(string $key): mixed {
        return $this->store[$key] ?? null;
    }

    public function set(string $key, mixed $value, int $ttl = 3600): void {
        $this->store[$key] = $value;
    }

    public function has(string $key): bool {
        return isset($this->store[$key]);
    }

    public function getAll(): array {
        return $this->store;
    }
}

class ProductService {
    public function __construct(
        private ProductRepository $repo,
        private Cache $cache
    ) {}

    public function getProduct(int $id): ?Product {
        $key = "product:$id";
        if ($this->cache->has($key)) {
            return $this->cache->get($key);
        }
        $product = $this->repo->find($id);
        if ($product) {
            $this->cache->set($key, $product);
        }
        return $product;
    }
}
?>
```


## Resources

- [Test Doubles](https://martinfowler.com/bliki/TestDouble.html) — Martin Fowler on Test Doubles

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*