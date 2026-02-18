---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-ddd-concepts"
---

# Domain-Driven Design Basics

## Introduction

Domain-Driven Design (DDD) focuses software development on the core business domain. It provides patterns for modeling complex business logic and organizing code around business concepts rather than technical layers.

## Key Concepts

- **Entity**: Object with identity that persists over time
- **Value Object**: Immutable object defined by its attributes
- **Aggregate**: Cluster of entities with a root entity
- **Domain Service**: Business logic that doesn't belong to an entity
- **Bounded Context**: Explicit boundary within which a model applies

## Real World Context

DDD helps when:
- Business logic is complex
- Domain experts are available
- The system will evolve over time
- Multiple teams work on the same codebase

## Deep Dive

### Entity with Identity

```typescript
export abstract class Entity<T> {
  protected readonly _id: T;

  get id(): T {
    return this._id;
  }

  equals(entity?: Entity<T>): boolean {
    if (entity === null || entity === undefined) return false;
    if (this === entity) return true;
    return this._id === entity._id;
  }
}

export class User extends Entity<string> {
  private _email: string;
  private _name: string;
  private _isActive: boolean;

  private constructor(id: string, email: string, name: string) {
    super();
    this._id = id;
    this._email = email;
    this._name = name;
    this._isActive = true;
  }

  static create(email: string, name: string): User {
    const id = uuid();
    return new User(id, email, name);
  }

  deactivate(): void {
    this._isActive = false;
  }

  get email(): string { return this._email; }
  get name(): string { return this._name; }
  get isActive(): boolean { return this._isActive; }
}
```

### Value Object (Immutable)

```typescript
export abstract class ValueObject<T> {
  protected readonly props: T;

  protected constructor(props: T) {
    this.props = Object.freeze(props);
  }

  equals(vo?: ValueObject<T>): boolean {
    if (vo === null || vo === undefined) return false;
    return JSON.stringify(this.props) === JSON.stringify(vo.props);
  }
}

export class Money extends ValueObject<{ amount: number; currency: string }> {
  private constructor(amount: number, currency: string) {
    super({ amount, currency });
  }

  static create(amount: number, currency: string): Money {
    if (amount < 0) throw new Error('Amount cannot be negative');
    return new Money(amount, currency);
  }

  add(money: Money): Money {
    if (this.props.currency !== money.props.currency) {
      throw new Error('Cannot add different currencies');
    }
    return Money.create(this.props.amount + money.props.amount, this.props.currency);
  }

  get amount(): number { return this.props.amount; }
  get currency(): string { return this.props.currency; }
}
```

### Aggregate Root

```typescript
export class Order extends Entity<string> {
  private _items: OrderItem[] = [];
  private _status: OrderStatus;
  private _total: Money;

  static create(customerId: string): Order {
    const order = new Order(uuid());
    order._status = OrderStatus.DRAFT;
    order._total = Money.create(0, 'USD');
    return order;
  }

  addItem(productId: string, quantity: number, price: Money): void {
    if (this._status !== OrderStatus.DRAFT) {
      throw new Error('Cannot modify non-draft order');
    }
    
    const item = OrderItem.create(productId, quantity, price);
    this._items.push(item);
    this.recalculateTotal();
  }

  submit(): void {
    if (this._items.length === 0) {
      throw new Error('Cannot submit empty order');
    }
    this._status = OrderStatus.SUBMITTED;
  }

  private recalculateTotal(): void {
    this._total = this._items.reduce(
      (sum, item) => sum.add(item.subtotal),
      Money.create(0, 'USD'),
    );
  }
}
```

### Domain Service

```typescript
@Injectable()
export class PricingService {
  calculateDiscount(order: Order, customer: Customer): Money {
    let discount = Money.create(0, order.total.currency);
    
    // Loyalty discount
    if (customer.orderCount > 10) {
      discount = discount.add(
        Money.create(order.total.amount * 0.1, order.total.currency),
      );
    }
    
    // Bulk discount
    if (order.itemCount > 5) {
      discount = discount.add(
        Money.create(order.total.amount * 0.05, order.total.currency),
      );
    }
    
    return discount;
  }
}
```

## Common Pitfalls

1. **Anemic domain models**: Entities with only getters/setters. Put behavior in entities.
2. **DDD everywhere**: Not all code needs DDD. Use for complex domains.
3. **Ignoring bounded contexts**: Different contexts may model the same concept differently.

## Best Practices

- Put business logic in entities and value objects
- Use value objects for concepts without identity
- Aggregate roots enforce invariants
- Domain services for cross-entity logic
- Ubiquitous language: use domain terms in code

## Summary

DDD organizes code around business concepts. Entities have identity, value objects are immutable. Aggregates ensure consistency, domain services handle cross-entity logic. Use DDD for complex business domains.

## Resources

- [CQRS and DDD](https://docs.nestjs.com/recipes/cqrs) — Domain patterns in NestJS

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*