---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-abstract-classes-mixins"
---

# Abstract Classes & Mixins

## Introduction
Abstract classes define a contract that subclasses must fulfill while also providing shared implementation. Mixins extend this idea by composing behaviors from multiple sources into a single class — something TypeScript supports through a pattern of class-returning functions.

## Key Concepts
- **Abstract Class**: A class that cannot be instantiated directly. It may contain both abstract methods (no body, must be overridden) and concrete methods (with implementation).
- **Mixin**: A function that takes a base class constructor and returns a new class extending it with additional functionality.
- **Constructor Type**: The `new (...args: any[]) => T` signature used to type class constructors in mixin functions.

## Real World Context
Frameworks like Angular and NestJS use abstract classes for base controllers and services. Mixins are used in libraries like Lit (web components) and TypeORM to compose behaviors such as timestamping, soft-deleting, and auditing without deep inheritance hierarchies.

## Deep Dive
### Abstract Classes

```typescript
abstract class Shape {
    abstract area(): number;
    abstract perimeter(): number;

    // Concrete method — shared by all subclasses
    describe(): string {
        return `Area: ${this.area().toFixed(2)}, Perimeter: ${this.perimeter().toFixed(2)}`;
    }
}

class Circle extends Shape {
    constructor(private radius: number) { super(); }
    area(): number { return Math.PI * this.radius ** 2; }
    perimeter(): number { return 2 * Math.PI * this.radius; }
}

class Rectangle extends Shape {
    constructor(private width: number, private height: number) { super(); }
    area(): number { return this.width * this.height; }
    perimeter(): number { return 2 * (this.width + this.height); }
}

// const s = new Shape(); // Error: Cannot create an instance of an abstract class
const c = new Circle(5);
console.log(c.describe()); // "Area: 78.54, Perimeter: 31.42"
```

Abstract classes combine the interface contract with shared implementation that subclasses inherit for free.

### Mixins

TypeScript models mixins as functions that accept a class constructor and return a new class:

```typescript
type Constructor<T = {}> = new (...args: any[]) => T;

function Timestamped<TBase extends Constructor>(Base: TBase) {
    return class extends Base {
        createdAt = new Date();
        updatedAt = new Date();

        touch() {
            this.updatedAt = new Date();
        }
    };
}

function SoftDeletable<TBase extends Constructor>(Base: TBase) {
    return class extends Base {
        deletedAt: Date | null = null;
        isDeleted = false;

        softDelete() {
            this.deletedAt = new Date();
            this.isDeleted = true;
        }
    };
}

// Compose mixins
class BaseEntity { id = crypto.randomUUID(); }

const TimestampedEntity = Timestamped(BaseEntity);
const FullEntity = SoftDeletable(TimestampedEntity);

const entity = new FullEntity();
entity.touch();       // from Timestamped
entity.softDelete();  // from SoftDeletable
console.log(entity.id); // from BaseEntity
```

Each mixin adds one behavior without creating a deep inheritance chain.

## Common Pitfalls
1. **Diamond problem** — If two mixins add a method with the same name, the last one applied wins. Be deliberate about composition order.
2. **Constructor signatures** — Mixins that add constructor parameters are more complex. Prefer mixins that use property initializers over constructor arguments.

## Best Practices
1. **Keep mixins focused** — Each mixin should add exactly one concern (timestamps, soft-delete, auditing). This keeps composition simple and predictable.
2. **Use abstract classes for "is-a" relationships** — If the hierarchy models a taxonomy (Shape → Circle), use abstract classes. If you are adding cross-cutting concerns, use mixins.

## Summary
- Abstract classes combine interface contracts with shared implementation.
- Mixins compose behaviors by chaining class-returning functions.
- Use abstract classes for taxonomies and mixins for cross-cutting concerns.
- Be mindful of method name collisions in mixin composition order.

## Code Examples

**A Serializable mixin that adds a serialize() method to any class — composed without inheritance**

```typescript
type Constructor<T = {}> = new (...args: any[]) => T;

// Mixin that adds serialization
function Serializable<TBase extends Constructor>(Base: TBase) {
    return class extends Base {
        serialize(): string {
            return JSON.stringify(this);
        }
    };
}

class User {
    constructor(public name: string, public email: string) {}
}

const SerializableUser = Serializable(User);
const user = new SerializableUser("Alice", "alice@example.com");
console.log(user.serialize());
// '{"name":"Alice","email":"alice@example.com"}'
```


## Resources

- [TypeScript Handbook: Mixins](https://www.typescriptlang.org/docs/handbook/mixins.html) — Official TypeScript documentation on the mixin pattern

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*