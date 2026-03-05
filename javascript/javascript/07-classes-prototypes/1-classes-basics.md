---
source_course: "javascript"
source_lesson: "javascript-classes-basics"
---

# ES6 Classes

## Introduction

ES6 classes provide a cleaner syntax for creating objects and implementing inheritance. While they're syntactic sugar over JavaScript's prototype system, they make object-oriented patterns more accessible and code more readable.

## Key Concepts

**Class**: A template for creating objects with shared methods and structure.

**Constructor**: A special method for initializing new instances.

**Instance**: An object created from a class using `new`.

## Real World Context

UI components, data models, game entities, API clients—classes organize complex codebases. React class components (legacy), TypeScript, and many libraries use classes extensively.

## Deep Dive

### Basic Class Syntax

```javascript
class User {
  // Constructor - called when using 'new'
  constructor(name, email) {
    this.name = name;
    this.email = email;
    this.createdAt = new Date();
  }
  
  // Instance method
  greet() {
    return `Hello, I'm ${this.name}`;
  }
  
  // Getter
  get displayName() {
    return `${this.name} <${this.email}>`;
  }
  
  // Setter
  set displayName(value) {
    const [name, email] = value.split(' <');
    this.name = name;
    this.email = email.replace('>', '');
  }
}

const user = new User('Alice', 'alice@example.com');
user.greet();        // 'Hello, I'm Alice'
user.displayName;    // 'Alice <alice@example.com>'
```

### Static Members

```javascript
class MathUtils {
  static PI = 3.14159;
  
  static square(n) {
    return n * n;
  }
  
  static cube(n) {
    return n * n * n;
  }
}

// Called on class, not instances
MathUtils.PI;        // 3.14159
MathUtils.square(4); // 16

const m = new MathUtils();
// m.square(4);  // Error - static methods aren't on instances
```

### Public and Private Fields (ES2022)

```javascript
class BankAccount {
  // Public field
  owner;
  
  // Private field (# prefix)
  #balance = 0;
  
  constructor(owner, initialBalance) {
    this.owner = owner;
    this.#balance = initialBalance;
  }
  
  deposit(amount) {
    if (amount > 0) this.#balance += amount;
  }
  
  get balance() {
    return this.#balance;  // Controlled access
  }
  
  // Private method
  #validateTransaction(amount) {
    return amount > 0 && amount <= this.#balance;
  }
}

const account = new BankAccount('Alice', 100);
account.balance;      // 100
// account.#balance;  // SyntaxError - private!
```

### Class Expressions

```javascript
// Named class expression
const MyClass = class ClassName {
  constructor() { /* ... */ }
};

// Anonymous class expression
const AnonClass = class {
  constructor() { /* ... */ }
};
```

## Common Pitfalls

1. **Forgetting `new`**: Classes throw without `new` (unlike constructor functions).
2. **Arrow methods for `this`**: Class methods aren't auto-bound; use arrows in constructor or bind.
3. **Private fields require declaration**: `#field` must be declared in class body.

## Best Practices

- **Use private fields for internal state**: Encapsulation prevents external manipulation.
- **Use getters for computed properties**: Instead of method calls.
- **Use static for utility methods**: That don't need instance data.
- **Keep classes focused**: Single responsibility principle.

## Summary

Classes provide a clean syntax for object-oriented JavaScript. Use `constructor` for initialization, define methods in the class body, and use `static` for class-level members. ES2022's private fields (`#`) enable true encapsulation. Always use `new` when instantiating classes.

## Code Examples

**Basic Class Syntax**

```javascript
class User {
  // Constructor - called when using 'new'
  constructor(name, email) {
    this.name = name;
    this.email = email;
    this.createdAt = new Date();
  }
  
  // Instance method
  greet() {
    return `Hello, I'm ${this.name}`;
  }
  
  // Getter
  get displayName() {
    return `${this.name} <${this.email}>`;
  }
```

**Static Members**

```javascript
class MathUtils {
  static PI = 3.14159;
  
  static square(n) {
    return n * n;
  }
  
  static cube(n) {
    return n * n * n;
  }
}

// Called on class, not instances
MathUtils.PI;        // 3.14159
MathUtils.square(4); // 16

const m = new MathUtils();
// m.square(4);  // Error - static methods aren't on instances
```


## Resources

- [MDN: Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes) — Complete class reference
- [MDN: Private class features](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields) — Private fields and methods

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*