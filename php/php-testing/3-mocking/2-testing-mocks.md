---
source_course: "php-testing"
source_lesson: "php-testing-mocks"
---

# Mocks with Expectations

Mocks verify that methods are called with specific arguments.

## Basic Mock

```php
<?php
public function testSendsWelcomeEmail(): void
{
    // Create mock with expectations
    $mailer = $this->createMock(Mailer::class);
    
    // Expect send() to be called exactly once
    $mailer->expects($this->once())
        ->method('send')
        ->with(
            $this->equalTo('john@example.com'),
            $this->equalTo('Welcome!'),
            $this->stringContains('Thank you for registering')
        );
    
    $service = new UserService($mailer);
    $service->register('john@example.com', 'password123');
    
    // Mock automatically verifies expectations after test
}
```

## Expectation Matchers

```php
<?php
$mock->expects($this->once())         // Exactly once
$mock->expects($this->never())        // Never called
$mock->expects($this->atLeastOnce())  // 1 or more
$mock->expects($this->exactly(3))     // Exactly 3 times
$mock->expects($this->atMost(2))      // 0, 1, or 2 times
$mock->expects($this->any())          // Any number of times
```

## Argument Constraints

```php
<?php
public function testWithConstraints(): void
{
    $mock = $this->createMock(Logger::class);
    
    $mock->expects($this->once())
        ->method('log')
        ->with(
            $this->identicalTo('error'),             // Strict comparison
            $this->stringContains('failed'),         // Contains substring
            $this->arrayHasKey('timestamp'),         // Array has key
            $this->isInstanceOf(Exception::class),   // Type check
            $this->greaterThan(0),                   // Numeric comparison
            $this->matchesRegularExpression('/\d+/'), // Regex
            $this->callback(fn($arg) => $arg > 10)   // Custom callback
        );
}
```

## Consecutive Calls

```php
<?php
public function testConsecutiveCalls(): void
{
    $generator = $this->createMock(IdGenerator::class);
    
    $generator->expects($this->exactly(3))
        ->method('generate')
        ->willReturnOnConsecutiveCalls(1, 2, 3);
    
    $service = new ItemService($generator);
    $item1 = $service->create('First');   // Uses ID 1
    $item2 = $service->create('Second');  // Uses ID 2
    $item3 = $service->create('Third');   // Uses ID 3
    
    $this->assertEquals(1, $item1->id);
    $this->assertEquals(2, $item2->id);
    $this->assertEquals(3, $item3->id);
}
```

## Verifying Call Order

```php
<?php
public function testCallOrder(): void
{
    $mock = $this->createMock(PaymentProcessor::class);
    
    // Define expected order
    $mock->expects($this->exactly(3))
        ->method('process')
        ->withConsecutive(
            [$this->equalTo('validate')],
            [$this->equalTo('charge')],
            [$this->equalTo('confirm')]
        )
        ->willReturnOnConsecutiveCalls(true, true, true);
    
    $service = new CheckoutService($mock);
    $service->checkout($order);
}
```

## Code Examples

**Complete mock example with payment and notification services**

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

interface PaymentGateway {
    public function charge(string $customerId, float $amount): string;
    public function refund(string $transactionId): bool;
}

interface NotificationService {
    public function sendEmail(string $to, string $subject, string $body): void;
    public function sendSms(string $phone, string $message): void;
}

class OrderService {
    public function __construct(
        private PaymentGateway $payment,
        private NotificationService $notifications
    ) {}
    
    public function processOrder(Order $order): string {
        $transactionId = $this->payment->charge(
            $order->customerId,
            $order->total
        );
        
        $this->notifications->sendEmail(
            $order->customerEmail,
            'Order Confirmed',
            "Your order #{$order->id} has been processed."
        );
        
        return $transactionId;
    }
}

class OrderServiceTest extends TestCase {
    public function testProcessOrderChargesAndNotifies(): void {
        // Arrange: Create mocks
        $payment = $this->createMock(PaymentGateway::class);
        $notifications = $this->createMock(NotificationService::class);
        
        $order = new Order(
            id: 123,
            customerId: 'cust_456',
            customerEmail: 'john@example.com',
            total: 99.99
        );
        
        // Set expectations on payment mock
        $payment->expects($this->once())
            ->method('charge')
            ->with(
                $this->equalTo('cust_456'),
                $this->equalTo(99.99)
            )
            ->willReturn('txn_789');
        
        // Set expectations on notification mock
        $notifications->expects($this->once())
            ->method('sendEmail')
            ->with(
                $this->equalTo('john@example.com'),
                $this->equalTo('Order Confirmed'),
                $this->stringContains('123')
            );
        
        // Act
        $service = new OrderService($payment, $notifications);
        $result = $service->processOrder($order);
        
        // Assert
        $this->assertEquals('txn_789', $result);
    }
    
    public function testProcessOrderFailsOnPaymentError(): void {
        $payment = $this->createMock(PaymentGateway::class);
        $notifications = $this->createMock(NotificationService::class);
        
        $order = new Order(id: 123, customerId: 'cust_456', customerEmail: 'john@example.com', total: 99.99);
        
        // Payment will fail
        $payment->expects($this->once())
            ->method('charge')
            ->willThrowException(new PaymentException('Card declined'));
        
        // Email should NOT be sent
        $notifications->expects($this->never())
            ->method('sendEmail');
        
        $service = new OrderService($payment, $notifications);
        
        $this->expectException(PaymentException::class);
        $service->processOrder($order);
    }
}
?>
```


---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*