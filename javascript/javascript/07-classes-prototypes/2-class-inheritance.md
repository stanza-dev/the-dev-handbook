---
source_course: "javascript"
source_lesson: "javascript-class-inheritance"
---

# Class Inheritance

## Introduction

Inheritance lets classes build upon other classes—sharing code while allowing specialization. JavaScript's `extends` keyword makes inheritance straightforward, while `super` provides access to parent functionality.

## Key Concepts

**extends**: Keyword to create a subclass that inherits from a parent.

**super**: Reference to the parent class (call constructor or methods).

**Method Overriding**: Redefining inherited methods in a subclass.

## Real World Context

Component hierarchies, error types, specialized versions of base classes—inheritance models "is-a" relationships. `TypeError extends Error`, `AdminUser extends User`, `Button extends Component`.

## Deep Dive

### Basic Inheritance

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    console.log(`${this.name} makes a sound.`);
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);  // MUST call super() first!
    this.breed = breed;
  }
  
  // Override parent method
  speak() {
    console.log(`${this.name} barks!`);
  }
  
  // New method
  fetch() {
    console.log(`${this.name} fetches the ball.`);
  }
}

const dog = new Dog('Rex', 'German Shepherd');
dog.speak();  // 'Rex barks!'
dog.fetch();  // 'Rex fetches the ball.'
dog instanceof Dog;    // true
dog instanceof Animal; // true
```

### Calling Parent Methods with super

```javascript
class Cat extends Animal {
  speak() {
    super.speak();  // Call parent's speak()
    console.log(`${this.name} also meows.`);
  }
}

const cat = new Cat('Whiskers');
cat.speak();
// 'Whiskers makes a sound.'
// 'Whiskers also meows.'
```

### Static Inheritance

```javascript
class Parent {
  static greet() {
    return 'Hello from Parent';
  }
}

class Child extends Parent {
  static greet() {
    return `${super.greet()} and Child`;
  }
}

Child.greet(); // 'Hello from Parent and Child'
```

### Extending Built-ins

```javascript
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}

class TypedArray extends Array {
  constructor(type, ...items) {
    super(...items);
    this.type = type;
  }
  
  push(...items) {
    items.forEach(item => {
      if (typeof item !== this.type) {
        throw new TypeError(`Expected ${this.type}`);
      }
    });
    return super.push(...items);
  }
}

const numbers = new TypedArray('number', 1, 2, 3);
numbers.push(4);      // OK
// numbers.push('5'); // TypeError
```

### Mixin Pattern (Composition over Inheritance)

```javascript
// Mixins - share behavior without inheritance
const Serializable = (Base) => class extends Base {
  toJSON() {
    return JSON.stringify(this);
  }
  
  static fromJSON(json) {
    return Object.assign(new this(), JSON.parse(json));
  }
};

const Timestamped = (Base) => class extends Base {
  constructor(...args) {
    super(...args);
    this.createdAt = new Date();
  }
};

// Apply mixins
class User extends Serializable(Timestamped(Object)) {
  constructor(name) {
    super();
    this.name = name;
  }
}

const user = new User('Alice');
user.toJSON();     // Works!
user.createdAt;    // Works!
```

## Common Pitfalls

1. **Forgetting `super()` in constructor**: Required before using `this`.
2. **Deep inheritance hierarchies**: Hard to understand and maintain.
3. **Overusing inheritance**: Prefer composition for flexibility.

## Best Practices

- **Call `super()` first in constructors**: Before any `this` usage.
- **Prefer shallow hierarchies**: 2-3 levels max.
- **Use composition over inheritance**: When possible.
- **Follow Liskov Substitution**: Subclasses should be usable as parent class.

## Summary

`extends` creates a subclass. `super()` must be called first in derived constructors. Use `super.method()` to call parent methods. Static methods are also inherited. Consider mixins for sharing behavior across unrelated classes. Keep hierarchies shallow.

## Resources

- [MDN: extends](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/extends) — extends keyword reference
- [MDN: super](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/super) — super keyword reference

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*