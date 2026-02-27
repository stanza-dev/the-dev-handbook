---
source_course: "javascript"
source_lesson: "javascript-this-keyword"
---

# The 'this' Keyword

## Introduction

The `this` keyword is one of JavaScript's most confusing aspects. Unlike other languages where `this` always refers to the current instance, JavaScript's `this` is determined by how a function is called. Understanding `this` is essential for working with objects, classes, and callbacks.

## Key Concepts

**this**: A keyword that refers to the context in which a function is executed.

**Implicit Binding**: When `this` is set by how a function is called.

**Explicit Binding**: When you manually set `this` using `call`, `apply`, or `bind`.

## Real World Context

Event handlers, object methods, class methods, and callbacks all involve `this`. Understanding how it works prevents bugs like "undefined is not a function" and enables proper object-oriented JavaScript.

## Deep Dive

### Default Binding

```javascript
// In browser global scope
function showThis() {
  console.log(this);
}
showThis();  // window (or undefined in strict mode)

// Strict mode
'use strict';
function strictThis() {
  console.log(this);
}
strictThis();  // undefined
```

### Implicit Binding (Method Calls)

```javascript
const user = {
  name: 'Alice',
  greet() {
    console.log(`Hello, I'm ${this.name}`);
  }
};

user.greet();  // 'Hello, I'm Alice'

// Lost binding!
const greet = user.greet;
greet();  // 'Hello, I'm undefined' (or error in strict)
```

### Explicit Binding

```javascript
function introduce(greeting, punctuation) {
  console.log(`${greeting}, I'm ${this.name}${punctuation}`);
}

const person = { name: 'Alice' };

// call - invokes immediately with arguments list
introduce.call(person, 'Hi', '!');  // 'Hi, I'm Alice!'

// apply - invokes immediately with arguments array
introduce.apply(person, ['Hello', '.']);  // 'Hello, I'm Alice.'

// bind - returns new function with bound this
const boundIntro = introduce.bind(person, 'Hey');
boundIntro('?');  // 'Hey, I'm Alice?'
```

### `new` Binding

```javascript
function Person(name) {
  // `this` refers to newly created object
  this.name = name;
}

const alice = new Person('Alice');
console.log(alice.name);  // 'Alice'
```

### Arrow Functions and `this`

```javascript
const obj = {
  name: 'Object',
  
  // Regular function: this = obj
  regular() {
    console.log(this.name);  // 'Object'
  },
  
  // Arrow function: this = enclosing scope (not obj)
  arrow: () => {
    console.log(this.name);  // undefined (global)
  },
  
  // Useful pattern: arrow in regular method
  delayedGreet() {
    setTimeout(() => {
      console.log(this.name);  // 'Object' (inherited)
    }, 100);
  }
};
```

### Binding Priority

1. **new** binding (highest)
2. Explicit binding (`call`, `apply`, `bind`)
3. Implicit binding (method call)
4. Default binding (global/undefined)

```javascript
function foo() {
  console.log(this.a);
}

const obj1 = { a: 1, foo };
const obj2 = { a: 2 };

obj1.foo();               // 1 (implicit)
obj1.foo.call(obj2);      // 2 (explicit overrides implicit)
new obj1.foo();           // undefined (new overrides implicit)
```

## Common Pitfalls

1. **Losing `this` in callbacks**: `setTimeout(user.method, 100)` loses binding.
2. **Arrow functions in objects**: Don't have their own `this`.
3. **Forgetting `new`**: Without `new`, constructor functions pollute global scope.

## Best Practices

- **Use arrow functions in callbacks**: To preserve `this` from outer scope.
- **Use `bind` for event handlers**: Or arrow functions in class properties.
- **Avoid `this` in nested functions**: Use arrow functions or capture in variable.
- **Use classes**: For clearer `this` semantics.

## Summary

`this` is determined by how a function is called, not where it's defined. Regular functions get `this` from their caller; arrow functions inherit from their lexical scope. Use `call`, `apply`, or `bind` for explicit control. The `new` keyword creates a new object and sets `this` to it.

## Resources

- [MDN: this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this) — Complete this keyword reference
- [MDN: Function.prototype.bind()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) — Creating bound functions

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*