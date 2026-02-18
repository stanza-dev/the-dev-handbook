---
source_course: "php-testing"
source_lesson: "php-testing-test-doubles"
---

# Understanding Test Doubles

Test doubles replace real dependencies in tests to isolate the code under test.

## Types of Test Doubles

| Type | Purpose | Verifies Calls? |
|------|---------|----------------|
| Dummy | Fill parameter, never used | No |
| Stub | Returns preset values | No |
| Spy | Records interactions | After test |
| Mock | Pre-programmed expectations | Yes, during test |
| Fake | Working implementation | No |

## Creating a Stub

```php
<?php
class OrderServiceTest extends TestCase
{
    public function testCalculateTotal(): void
    {
        // Create stub
        $productRepository = $this->createStub(ProductRepository::class);
        
        // Configure return value
        $productRepository->method('findById')
            ->willReturn(new Product(id: 1, name: 'Widget', price: 10.00));
        
        $service = new OrderService($productRepository);
        $total = $service->calculateTotal([1, 1, 1]);  // 3 widgets
        
        $this->assertEquals(30.00, $total);
    }
}
```

## Stub with Multiple Returns

```php
<?php
public function testMultipleProducts(): void
{
    $repo = $this->createStub(ProductRepository::class);
    
    // Return different values for different arguments
    $repo->method('findById')
        ->willReturnMap([
            [1, new Product(1, 'Widget', 10.00)],
            [2, new Product(2, 'Gadget', 20.00)],
            [3, null],  // Not found
        ]);
    
    $service = new OrderService($repo);
    
    $this->assertEquals(30.00, $service->calculateTotal([1, 2]));
}
```

## Stub Returning Callback

```php
<?php
public function testDynamicReturn(): void
{
    $repo = $this->createStub(ProductRepository::class);
    
    $repo->method('findById')
        ->willReturnCallback(function (int $id) {
            return new Product($id, "Product $id", $id * 10.0);
        });
    
    $service = new OrderService($repo);
    $total = $service->calculateTotal([1, 2, 3]);
    
    // 10 + 20 + 30 = 60
    $this->assertEquals(60.00, $total);
}
```

## Stub Throwing Exceptions

```php
<?php
public function testHandlesNotFound(): void
{
    $repo = $this->createStub(ProductRepository::class);
    
    $repo->method('findById')
        ->willThrowException(new ProductNotFoundException());
    
    $service = new OrderService($repo);
    
    $this->expectException(OrderException::class);
    $service->calculateTotal([999]);
}
```

## Resources

- [Test Doubles](https://docs.phpunit.de/en/11.4/test-doubles.html) — PHPUnit test doubles documentation

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*