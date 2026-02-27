---
source_course: "javascript"
source_lesson: "javascript-prototypes"
---

# Prototypes & Prototype Chain

## Introduction

Under the hood, JavaScript's inheritance is prototype-based, not class-based. Every object has an internal link to another object called its prototype. Understanding prototypes reveals how JavaScript really works and explains behaviors that seem magical with classes.

## Key Concepts

**Prototype**: An object from which other objects inherit properties.

**Prototype Chain**: The chain of prototypes that JavaScript searches when accessing properties.

**`__proto__`**: The actual prototype link (prefer `Object.getPrototypeOf()`).

## Real World Context

Understanding prototypes helps debug inheritance issues, optimize performance, and work with legacy code. It's essential knowledge for JavaScript interviews and understanding how the language works.

## Deep Dive

### The Prototype Chain

```javascript
const obj = { a: 1 };

// obj -> Object.prototype -> null
obj.toString();  // Works! Inherited from Object.prototype

const arr = [1, 2, 3];
// arr -> Array.prototype -> Object.prototype -> null
arr.push(4);     // From Array.prototype
arr.toString();  // From Object.prototype

// Check prototype
Object.getPrototypeOf(arr) === Array.prototype;  // true
Array.prototype.isPrototypeOf(arr);              // true
```

### How Property Lookup Works

```javascript
const parent = { a: 1, b: 2 };
const child = Object.create(parent);
child.b = 20;  // Shadow parent's b
child.c = 3;

child.a;  // 1 (from parent)
child.b;  // 20 (own property, shadows parent's b)
child.c;  // 3 (own property)
child.d;  // undefined (not in chain)

// Check own property
child.hasOwnProperty('a');  // false
child.hasOwnProperty('b');  // true
Object.hasOwn(child, 'b');  // true (ES2022)
```

### Classes Are Prototype Sugar

```javascript
class Dog {
  constructor(name) {
    this.name = name;
  }
  bark() {
    console.log('Woof!');
  }
}

// Is equivalent to:
function DogFunc(name) {
  this.name = name;
}
DogFunc.prototype.bark = function() {
  console.log('Woof!');
};

// Both work the same way
const dog1 = new Dog('Rex');
const dog2 = new DogFunc('Max');

Object.getPrototypeOf(dog1) === Dog.prototype;       // true
Object.getPrototypeOf(dog2) === DogFunc.prototype;   // true
```

### Object.create()

```javascript
// Create object with specific prototype
const personProto = {
  greet() {
    return `Hello, I'm ${this.name}`;
  }
};

const alice = Object.create(personProto);
alice.name = 'Alice';
alice.greet();  // 'Hello, I'm Alice'

// Create object with no prototype
const nullProto = Object.create(null);
nullProto.toString;  // undefined (no Object.prototype!)

// Useful for dictionaries without inherited properties
const dict = Object.create(null);
dict['constructor'] = 'safe';  // No collision with Object.prototype
```

### Modifying Prototypes (Carefully!)

```javascript
// Add method to all arrays (DON'T DO THIS!)
Array.prototype.first = function() {
  return this[0];
};

[1, 2, 3].first();  // 1

// Better: Use Object.defineProperty for non-enumerable
Object.defineProperty(Array.prototype, 'last', {
  value: function() { return this[this.length - 1]; },
  enumerable: false,
  writable: true,
  configurable: true
});
```

### instanceof and Prototype Checking

```javascript
class Animal {}
class Dog extends Animal {}

const dog = new Dog();

dog instanceof Dog;    // true
dog instanceof Animal; // true
dog instanceof Object; // true

// Manual check
Animal.prototype.isPrototypeOf(dog);  // true
```

## Common Pitfalls

1. **Modifying built-in prototypes**: Can break other code; avoid.
2. **Using `__proto__` directly**: Use `Object.getPrototypeOf()` instead.
3. **for...in includes inherited properties**: Use `hasOwnProperty` check.

## Best Practices

- **Use classes for OOP**: They're cleaner and standardized.
- **Use `Object.create(null)` for dictionaries**: No inherited properties.
- **Don't modify built-in prototypes**: Except for polyfills.
- **Use `Object.hasOwn()` over `hasOwnProperty()`**: Safer for all objects.

## Summary

Every object has a prototype forming a chain. Property lookup traverses this chain. Classes are syntactic sugar over prototypes. `Object.create()` creates objects with custom prototypes. Avoid modifying built-in prototypes. Use `Object.hasOwn()` to check for own properties.

## Resources

- [MDN: Inheritance and the prototype chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain) — Deep dive into prototypes
- [MDN: Object.prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/prototype) — Object.prototype reference

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*