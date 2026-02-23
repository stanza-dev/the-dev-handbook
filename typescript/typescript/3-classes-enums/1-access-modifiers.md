---
source_course: "typescript"
source_lesson: "typescript-classes-access-modifiers"
---

# Classes & Access Modifiers

## Introduction
TypeScript enhances JavaScript classes with access modifiers, typed properties, and compile-time visibility checks. If you come from languages like Java or C#, these features will feel familiar. If you are coming from JavaScript, they add a layer of encapsulation that helps you design cleaner APIs.

## Key Concepts
- **Access Modifier**: A keyword (`public`, `private`, `protected`) that controls who can access a class member.
- **Parameter Properties**: A shorthand that declares and assigns constructor parameters as class properties in one step.
- **Class Fields**: Typed properties declared directly in the class body.

## Real World Context
In a backend application, you might have a `DatabaseConnection` class where the connection string is `private` (only the class itself can use it), a `query` method is `public` (available to consumers), and a `logQuery` method is `protected` (available to subclasses that add logging). Access modifiers enforce these boundaries at compile time.

## Deep Dive

### Public, Private, and Protected

Every class member is `public` by default:

```typescript
class UserAccount {
  public name: string;       // Accessible everywhere
  private password: string;  // Only within UserAccount
  protected email: string;   // Within UserAccount and subclasses

  constructor(name: string, password: string, email: string) {
    this.name = name;
    this.password = password;
    this.email = email;
  }

  public getProfile(): string {
    return `${this.name} (${this.email})`;
  }

  private hashPassword(): string {
    return `hashed_${this.password}`;
  }
}
```

The `password` field is invisible outside the class, and `hashPassword` cannot be called by external code. This prevents sensitive data from leaking through the class API.

### Subclasses and Protected Members

`protected` members are accessible in subclasses but not outside:

```typescript
class AdminAccount extends UserAccount {
  private adminLevel: number;

  constructor(name: string, password: string, email: string, level: number) {
    super(name, password, email);
    this.adminLevel = level;
  }

  getAdminProfile(): string {
    // Can access protected 'email' from parent
    return `Admin ${this.name} (${this.email}) - Level ${this.adminLevel}`;
    // Cannot access private 'password' from parent
    // this.password; // Error
  }
}
```

The `AdminAccount` can read `email` because it is `protected`, but it cannot read `password` because it is `private` to `UserAccount`.

### Parameter Properties

TypeScript offers a shorthand to declare and assign properties directly in the constructor:

```typescript
class Product {
  constructor(
    public name: string,
    public price: number,
    private sku: string
  ) {}
  // No need to write this.name = name, etc.
}

const item = new Product("Keyboard", 79.99, "KB-001");
console.log(item.name);  // "Keyboard"
// item.sku; // Error: 'sku' is private
```

This eliminates boilerplate and keeps the class concise.

## Common Pitfalls
1. **Relying on `private` for security** — TypeScript's `private` is a compile-time check only. At runtime, JavaScript has no access modifiers (unless you use `#private` fields). Do not use `private` for true security; use it for API design.
2. **Forgetting to call `super()`** — Subclass constructors must call `super()` before accessing `this`. TypeScript enforces this, but it is a common source of confusion.

## Best Practices
1. **Use parameter properties for simple classes** — They reduce boilerplate significantly. When a constructor just assigns parameters to properties, parameter properties are the cleanest approach.
2. **Default to `private`, expose as needed** — Start with the most restrictive visibility and widen it only when external access is genuinely required.

## Summary
- `public` members are accessible everywhere, `private` only within the class, and `protected` within the class and its subclasses.
- Parameter properties combine declaration and assignment in the constructor signature.
- Access modifiers are compile-time only and serve as API design tools, not runtime security.

## Code Examples

**Parameter properties declare, assign, and control visibility of class members in the constructor signature**

```typescript
// Parameter properties eliminate boilerplate
class OrderItem {
  constructor(
    public readonly productName: string,
    public quantity: number,
    private unitPrice: number
  ) {}

  getTotal(): number {
    return this.quantity * this.unitPrice;
  }
}

const item = new OrderItem("Mechanical Keyboard", 2, 89.99);
console.log(item.getTotal()); // 179.98
console.log(item.productName); // "Mechanical Keyboard"
// item.unitPrice; // Error: 'unitPrice' is private
```


## Resources

- [TypeScript Handbook: Classes](https://www.typescriptlang.org/docs/handbook/2/classes.html) — Official guide to classes, access modifiers, and class features in TypeScript

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*