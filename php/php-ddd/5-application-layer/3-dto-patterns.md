---
source_course: "php-ddd"
source_lesson: "php-ddd-dto-patterns"
---

# Data Transfer Objects in DDD

## Introduction

Data Transfer Objects (DTOs) are simple objects used to carry data across boundaries in a DDD application. They decouple the internal domain model from external consumers, preventing implementation details from leaking through APIs and ensuring each layer communicates with appropriately shaped data.

## Key Concepts

- **DTO** - a plain object that carries data between processes or layers without behavior
- **Command Object** - a DTO representing an intent to change state
- **Query Object** - a DTO representing a request for data
- **Assembler / Mapper** - a component that converts between domain objects and DTOs

## Real World Context

In a typical web application, a controller receives an HTTP request, maps it to a command DTO, passes it to an application service, and then maps the domain result back to a response DTO. This layered transformation keeps the domain model insulated from HTTP concerns and allows API formats to evolve independently.

## Deep Dive

### Readonly DTOs in PHP 8.5

PHP 8.5 readonly classes provide a natural fit for DTOs since they are immutable after construction and have concise syntax.

```php
<?php
declare(strict_types=1);

namespace Application\Order\DTO;

final readonly class CreateOrderCommand
{
    /**
     * @param OrderItemInput[] $items
     */
    public function __construct(
        public string $customerId,
        public array $items,
        public ?string $couponCode = null,
    ) {}
}

final readonly class OrderItemInput
{
    public function __construct(
        public string $productId,
        public int $quantity,
    ) {}
}
```

### Command Objects

Commands represent an intent to perform a write operation. They carry only the data needed for the use case and validate basic structural constraints.

```php
<?php
namespace Application\Customer\Command;

final readonly class RegisterCustomerCommand
{
    public function __construct(
        public string $email,
        public string $name,
        public string $password,
    ) {
        if (empty($this->email) || empty($this->name)) {
            throw new \InvalidArgumentException('Email and name are required.');
        }
    }
}

final readonly class ChangeCustomerAddressCommand
{
    public function __construct(
        public string $customerId,
        public string $street,
        public string $city,
        public string $postalCode,
        public string $country,
    ) {}
}
```

### Query Objects

Queries represent a request for data. They carry criteria and pagination parameters but never trigger side effects.

```php
<?php
namespace Application\Order\Query;

final readonly class ListOrdersQuery
{
    public function __construct(
        public ?string $customerId = null,
        public ?string $status = null,
        public int $page = 1,
        public int $perPage = 20,
    ) {}
}

final readonly class GetOrderDetailQuery
{
    public function __construct(
        public string $orderId,
    ) {}
}
```

### Response DTOs

Response DTOs shape the data returned to the caller. They are independent of the domain model structure.

```php
<?php
namespace Application\Order\DTO;

final readonly class OrderResponse
{
    /**
     * @param OrderLineResponse[] $lines
     */
    public function __construct(
        public string $id,
        public string $status,
        public string $customerName,
        public string $total,
        public array $lines,
        public string $createdAt,
    ) {}
}

final readonly class OrderLineResponse
{
    public function __construct(
        public string $productName,
        public int $quantity,
        public string $unitPrice,
        public string $subtotal,
    ) {}
}
```

### Assemblers and Mappers

Assemblers convert between domain objects and DTOs. They live in the application layer and keep mapping logic centralized.

```php
<?php
namespace Application\Order\Assembler;

final class OrderAssembler
{
    public static function toResponse(Order $order, Customer $customer): OrderResponse
    {
        return new OrderResponse(
            id: $order->id()->toString(),
            status: $order->status()->value,
            customerName: $customer->name()->full(),
            total: $order->total()->format(),
            lines: array_map(
                fn(OrderLine $line) => new OrderLineResponse(
                    productName: $line->productName()->value(),
                    quantity: $line->quantity()->value(),
                    unitPrice: $line->unitPrice()->format(),
                    subtotal: $line->subtotal()->format(),
                ),
                $order->lines(),
            ),
            createdAt: $order->createdAt()->format('c'),
        );
    }

    public static function toDomain(CreateOrderCommand $cmd): array
    {
        return array_map(
            fn(OrderItemInput $item) => [
                'productId' => ProductId::fromString($item->productId),
                'quantity' => new Quantity($item->quantity),
            ],
            $cmd->items,
        );
    }
}
```

### Controller Integration

```php
<?php
namespace Infrastructure\Http\Controller;

final class OrderController
{
    public function __construct(
        private CommandBus $commandBus,
        private QueryBus $queryBus,
    ) {}

    public function create(Request $request): Response
    {
        $command = new CreateOrderCommand(
            customerId: $request->json('customer_id'),
            items: array_map(
                fn(array $i) => new OrderItemInput($i['product_id'], $i['quantity']),
                $request->json('items'),
            ),
            couponCode: $request->json('coupon_code'),
        );

        $orderId = $this->commandBus->dispatch($command);

        return new JsonResponse(['id' => $orderId], 201);
    }

    public function list(Request $request): Response
    {
        $query = new ListOrdersQuery(
            customerId: $request->query('customer_id'),
            status: $request->query('status'),
            page: (int) $request->query('page', '1'),
        );

        $result = $this->queryBus->ask($query);

        return new JsonResponse($result);
    }
}
```

## Common Pitfalls

1. **Exposing domain objects directly** - Returning entities from controllers couples clients to the internal model and makes refactoring dangerous. Always map to response DTOs.
2. **Putting business logic in DTOs** - DTOs should be pure data carriers. Validation in DTOs should be limited to structural constraints (required fields, type checks), not business rules.
3. **Creating too many DTO layers** - One command DTO and one response DTO per use case is usually enough. Avoid unnecessary intermediate DTO transformations that add complexity without benefit.

## Best Practices

1. **Use readonly classes for all DTOs** - Immutability prevents accidental modification after creation and clearly communicates intent.
2. **Name DTOs by intent** - Use names like `CreateOrderCommand` and `OrderResponse` that describe purpose, not generic names like `OrderDTO`.
3. **Co-locate DTOs with their use case** - Place command, query, and response DTOs alongside the handler that uses them in the application layer.

## Summary

- DTOs decouple the domain model from external consumers and transport mechanisms
- Command objects carry write intent, query objects carry read criteria
- Assemblers centralize the mapping between domain objects and DTOs
- PHP 8.5 readonly classes provide ideal DTO implementations
- Keep DTOs as pure data carriers without business logic

## Resources

- [Data Transfer Object Pattern](https://martinfowler.com/eaaCatalog/dataTransferObject.html) — Martin Fowler's catalog entry on the DTO pattern

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*