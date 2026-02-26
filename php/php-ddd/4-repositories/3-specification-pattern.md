---
source_course: "php-ddd"
source_lesson: "php-ddd-specification-pattern"
---

# The Specification Pattern

## Introduction

The Specification Pattern encapsulates business rules into reusable, composable objects. It provides a clean way to express complex query criteria and validation rules without polluting repositories or domain objects with conditional logic.

## Key Concepts

- **Specification** - a single business rule encapsulated as an object
- **Composite Specification** - combining specifications with AND, OR, and NOT operators
- **Candidate** - the object being evaluated against a specification
- **Repository Integration** - using specifications to build database queries dynamically

## Real World Context

E-commerce platforms use specifications to filter products by complex criteria such as price ranges, availability, customer eligibility, and promotional rules. Each rule is a separate specification that can be composed at runtime based on the user's request.

## Deep Dive

### Basic Specification Interface

```php
<?php
declare(strict_types=1);

namespace Domain\Shared\Specification;

/**
 * @template T
 */
interface Specification
{
    /**
     * @param T $candidate
     */
    public function isSatisfiedBy(mixed $candidate): bool;
}
```

### Concrete Specifications

```php
<?php
namespace Domain\Order\Specification;

final readonly class OrderIsOverdue implements Specification
{
    public function __construct(
        private \DateTimeImmutable $now
    ) {}

    public function isSatisfiedBy(mixed $candidate): bool
    {
        assert($candidate instanceof Order);

        return $candidate->status() === OrderStatus::Pending
            && $candidate->createdAt()->diff($this->now)->days > 7;
    }
}

final readonly class OrderExceedsAmount implements Specification
{
    public function __construct(
        private Money $threshold
    ) {}

    public function isSatisfiedBy(mixed $candidate): bool
    {
        assert($candidate instanceof Order);

        return $candidate->total()->isGreaterThan($this->threshold);
    }
}

final readonly class CustomerIsEligibleForDiscount implements Specification
{
    public function __construct(
        private int $minimumOrders
    ) {}

    public function isSatisfiedBy(mixed $candidate): bool
    {
        assert($candidate instanceof Customer);

        return $candidate->completedOrderCount() >= $this->minimumOrders;
    }
}
```

### Composite Specifications

```php
<?php
namespace Domain\Shared\Specification;

final readonly class AndSpecification implements Specification
{
    /** @var Specification[] */
    private array $specifications;

    public function __construct(Specification ...$specifications)
    {
        $this->specifications = $specifications;
    }

    public function isSatisfiedBy(mixed $candidate): bool
    {
        foreach ($this->specifications as $spec) {
            if (!$spec->isSatisfiedBy($candidate)) {
                return false;
            }
        }
        return true;
    }
}

final readonly class OrSpecification implements Specification
{
    /** @var Specification[] */
    private array $specifications;

    public function __construct(Specification ...$specifications)
    {
        $this->specifications = $specifications;
    }

    public function isSatisfiedBy(mixed $candidate): bool
    {
        foreach ($this->specifications as $spec) {
            if ($spec->isSatisfiedBy($candidate)) {
                return true;
            }
        }
        return false;
    }
}

final readonly class NotSpecification implements Specification
{
    public function __construct(
        private Specification $specification
    ) {}

    public function isSatisfiedBy(mixed $candidate): bool
    {
        return !$this->specification->isSatisfiedBy($candidate);
    }
}
```

### Fluent Composition

```php
<?php
namespace Domain\Shared\Specification;

abstract class ComposableSpecification implements Specification
{
    public function and(Specification $other): Specification
    {
        return new AndSpecification($this, $other);
    }

    public function or(Specification $other): Specification
    {
        return new OrSpecification($this, $other);
    }

    public function not(): Specification
    {
        return new NotSpecification($this);
    }
}

// Usage
$spec = (new OrderIsOverdue($now))
    ->and(new OrderExceedsAmount(Money::fromCents(10000)))
    ->and((new OrderIsCancelled())->not());

$overdueHighValueOrders = array_filter(
    $allOrders,
    fn(Order $order) => $spec->isSatisfiedBy($order)
);
```

### Repository Integration

Specifications can be translated to database queries so filtering happens at the database level rather than in memory.

```php
<?php
namespace Infrastructure\Persistence\Doctrine;

final class DoctrineOrderRepository implements OrderRepository
{
    public function findMatching(Specification $spec): array
    {
        $qb = $this->em->createQueryBuilder()
            ->select('o')
            ->from(Order::class, 'o');

        $visitor = new DoctrineSpecificationVisitor($qb);
        $visitor->visit($spec);

        return $qb->getQuery()->getResult();
    }
}

final class DoctrineSpecificationVisitor
{
    public function __construct(private QueryBuilder $qb) {}

    public function visit(Specification $spec): void
    {
        match (true) {
            $spec instanceof OrderIsOverdue => $this->visitOverdue($spec),
            $spec instanceof OrderExceedsAmount => $this->visitExceedsAmount($spec),
            $spec instanceof AndSpecification => $this->visitAnd($spec),
            $spec instanceof OrSpecification => $this->visitOr($spec),
            default => throw new \RuntimeException('Unsupported specification'),
        };
    }

    private function visitOverdue(OrderIsOverdue $spec): void
    {
        $this->qb
            ->andWhere('o.status = :status')
            ->andWhere('o.createdAt < :overdueDate')
            ->setParameter('status', OrderStatus::Pending->value)
            ->setParameter('overdueDate', $spec->thresholdDate());
    }

    private function visitExceedsAmount(OrderExceedsAmount $spec): void
    {
        $this->qb
            ->andWhere('o.totalCents > :threshold')
            ->setParameter('threshold', $spec->threshold()->cents());
    }
}
```

## Common Pitfalls

1. **Over-specifying simple queries** - If a query is used in only one place, a specification may add unnecessary abstraction. Reserve the pattern for rules that are reused or composed.
2. **Forgetting database translation** - In-memory evaluation works for small datasets but becomes a performance bottleneck when filtering large tables. Always provide a query-level implementation for database-backed repositories.
3. **Leaking infrastructure into specifications** - Specifications should express business rules using domain language, not SQL fragments or ORM-specific constructs.

## Best Practices

1. **Use readonly classes** - PHP 8.2+ readonly classes are ideal for specifications since they are inherently immutable and have clear constructor-based configuration.
2. **Compose at the application layer** - Build composite specifications in application services based on the use case, keeping individual specifications focused on a single rule.
3. **Test specifications independently** - Each specification should have its own unit test verifying the business rule it encapsulates.

## Summary

- The Specification Pattern encapsulates business rules as composable, reusable objects
- Specifications can be combined with AND, OR, and NOT operators for complex criteria
- Repository integration translates specifications into efficient database queries
- PHP readonly classes provide an ideal implementation for immutable specifications
- Keep specifications focused on domain rules, not infrastructure details

## Resources

- [Specification Pattern in DDD](https://martinfowler.com/apsupp/spec.pdf) — Martin Fowler and Eric Evans' original paper on the Specification pattern

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*