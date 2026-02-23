---
source_course: "typescript"
source_lesson: "typescript-abstract-classes-readonly"
---

# Abstract Classes and Readonly

## Introduction
TypeScript provides abstract classes for defining base classes that cannot be instantiated directly, and the `readonly` modifier for properties that should never change after initialization. Together, these features help you design safer class hierarchies and immutable data.

## Key Concepts
- **Abstract Class**: A class that cannot be instantiated directly and may contain abstract methods that subclasses must implement.
- **Abstract Method**: A method signature with no implementation, declared with the `abstract` keyword.
- **`readonly` Modifier**: A property modifier that prevents reassignment after the constructor completes.
- **Parameter Properties with Readonly**: Combining `readonly` with parameter properties for immutable fields.

## Real World Context
Abstract classes are common in payment processing: a base `PaymentProvider` class defines the interface, while `StripeProvider` and `PayPalProvider` implement the details. The `readonly` modifier is used for configuration values, entity IDs, and anything that should never change after creation.

## Deep Dive

### Abstract Classes

An abstract class defines a contract that subclasses must fulfill:

```typescript
abstract class Shape {
  abstract getArea(): number;
  abstract getPerimeter(): number;

  describe(): string {
    return `Area: ${this.getArea()}, Perimeter: ${this.getPerimeter()}`;
  }
}
```

The `Shape` class cannot be instantiated directly. It provides a concrete `describe()` method and requires subclasses to implement `getArea()` and `getPerimeter()`.

Subclasses provide the implementations:

```typescript
class Circle extends Shape {
  constructor(private radius: number) {
    super();
  }

  getArea(): number {
    return Math.PI * this.radius ** 2;
  }

  getPerimeter(): number {
    return 2 * Math.PI * this.radius;
  }
}

const circle = new Circle(5);
console.log(circle.describe()); // "Area: 78.54, Perimeter: 31.42"
// new Shape(); // Error: Cannot create an instance of an abstract class
```

Abstract classes are useful when you want to share behavior (like `describe()`) while enforcing a contract on subclasses.

### The `readonly` Modifier

The `readonly` modifier prevents reassignment after initialization:

```typescript
class Config {
  readonly apiUrl: string;
  readonly maxRetries: number;

  constructor(apiUrl: string, maxRetries: number) {
    this.apiUrl = apiUrl;
    this.maxRetries = maxRetries;
  }
}

const config = new Config("https://api.example.com", 3);
// config.apiUrl = "https://other.com"; // Error: Cannot assign to 'apiUrl' because it is a read-only property
```

The property can only be set in the constructor. After that, it is frozen.

### Combining Readonly with Parameter Properties

For maximum conciseness:

```typescript
class Invoice {
  constructor(
    public readonly id: string,
    public readonly amount: number,
    public readonly createdAt: Date
  ) {}
}

const invoice = new Invoice("INV-001", 250.00, new Date());
console.log(invoice.id); // "INV-001"
// invoice.amount = 300; // Error: Cannot assign to 'amount'
```

This pattern is ideal for value objects and entities where the data should be immutable after creation.

## Common Pitfalls
1. **Thinking `readonly` means deeply immutable** — `readonly` only prevents reassignment of the property itself. If the property is an object or array, its contents can still be mutated. Use `Readonly<T>` or `ReadonlyArray<T>` for deeper immutability.
2. **Overusing abstract classes** — In TypeScript, interfaces are often a better choice than abstract classes when you do not need shared implementation. Abstract classes create a rigid hierarchy; interfaces are more flexible.

## Best Practices
1. **Use `readonly` for entity IDs and configuration** — Any value that should not change after creation should be marked `readonly`. This prevents accidental mutations.
2. **Prefer interfaces over abstract classes when no shared implementation is needed** — Interfaces are lighter and allow multiple implementations without class hierarchy constraints.

## Summary
- Abstract classes define contracts with optional shared implementation.
- Subclasses must implement all abstract methods.
- The `readonly` modifier prevents property reassignment after construction.
- Combine `readonly` with parameter properties for concise, immutable class definitions.

## Code Examples

**An abstract base class that shares a log method while requiring subclasses to implement the send method**

```typescript
// Abstract class with shared logic and enforced contract
abstract class Notification {
  constructor(public readonly recipient: string) {}

  abstract send(): Promise<boolean>;

  log(): void {
    console.log(`Notification queued for ${this.recipient}`);
  }
}

class EmailNotification extends Notification {
  constructor(recipient: string, private subject: string) {
    super(recipient);
  }

  async send(): Promise<boolean> {
    console.log(`Sending email to ${this.recipient}: ${this.subject}`);
    return true;
  }
}
```


## Resources

- [TypeScript Handbook: Classes (Abstract)](https://www.typescriptlang.org/docs/handbook/2/classes.html#abstract-classes-and-members) — Official documentation on abstract classes and abstract members

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*