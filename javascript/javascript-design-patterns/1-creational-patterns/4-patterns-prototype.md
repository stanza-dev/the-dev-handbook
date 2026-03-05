---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-prototype"
---

# The Prototype Pattern

## Introduction

The Prototype pattern creates new objects by cloning existing ones. In JavaScript, this pattern is built into the language through prototypal inheritance. Understanding it helps you work with Object.create() and prototype chains.

## Key Concepts

**Prototype**: An object used as a template for new objects.

**Clone**: Creating a new object based on an existing one.

**Prototypal Inheritance**: JavaScript's native inheritance mechanism.

## Real World Context

JavaScript itself uses the Prototype pattern natively—every object has a prototype chain. Libraries like Lodash use `structuredClone()` (a Web API available since 2022) and spread operators for deep/shallow cloning. Component systems often clone template objects to stamp out new instances efficiently.

## Deep Dive

### Object.create()

The `Object.create()` method creates a new object with the specified prototype, letting us set up a clean prototype chain:

```javascript
const vehiclePrototype = {
  init(make, model) {
    this.make = make;
    this.model = model;
    return this;
  },
  getInfo() {
    return `${this.make} ${this.model}`;
  }
};

const car = Object.create(vehiclePrototype).init('Toyota', 'Camry');
car.getInfo();  // 'Toyota Camry'

// car's prototype IS vehiclePrototype
Object.getPrototypeOf(car) === vehiclePrototype;  // true
```

The `car` object delegates method lookups to `vehiclePrototype` through the prototype chain, so `getInfo()` is shared rather than copied.

### Clone with Customization

Prototypes enable efficient inheritance-based cloning where the clone inherits defaults from the original and only stores overridden values:

```javascript
const defaultConfig = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3,
  clone() {
    return Object.create(this);
  }
};

const customConfig = defaultConfig.clone();
customConfig.timeout = 10000;  // Override
customConfig.apiUrl;  // 'https://api.example.com' (inherited)
```

The `customConfig` only stores the overridden `timeout` property. The `apiUrl` lookup falls through to the prototype, keeping memory usage minimal.

### Deep Clone Pattern

When you need a fully independent copy rather than prototype-based inheritance, a recursive deep clone copies all nested objects:

```javascript
function deepClone(obj) {
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }
  
  if (Array.isArray(obj)) {
    return obj.map(deepClone);
  }
  
  const cloned = {};
  for (const key in obj) {
    if (Object.hasOwn(obj, key)) {
      cloned[key] = deepClone(obj[key]);
    }
  }
  return cloned;
}

// Modern: structuredClone()
const clone = structuredClone(original);
```

The manual `deepClone` handles nested objects and arrays recursively. For most use cases, the built-in `structuredClone()` is simpler and handles more edge cases like `Date`, `Map`, and `Set`.

### Registry of Prototypes

A prototype registry stores named prototypes in a Map, allowing you to stamp out new instances by name without knowing the concrete type:

```javascript
class ShapeRegistry {
  static #prototypes = new Map();
  
  static register(name, prototype) {
    this.#prototypes.set(name, prototype);
  }
  
  static create(name) {
    const proto = this.#prototypes.get(name);
    if (!proto) throw new Error(`Unknown shape: ${name}`);
    return Object.create(proto);
  }
}

ShapeRegistry.register('circle', {
  type: 'circle',
  draw() { console.log('Drawing circle'); }
});

const circle = ShapeRegistry.create('circle');
```

The registry decouples creation from specific prototype objects. Callers request a shape by name, and the registry handles cloning via `Object.create()`.

## Common Pitfalls

1. **Shallow vs deep clone**: Object.create/spread are shallow.
2. **Modifying prototype affects all clones**: Be careful with shared state.
3. **Performance with deep hierarchies**: Long prototype chains are slow.

## Best Practices

- **Use Object.create() for true prototypal patterns**: Not just spread.
- **Use structuredClone() for deep copies**: Modern and handles more types.
- **Keep prototype chains short**: For performance.
- **Prefer composition over inheritance**: When possible.

## Summary

The Prototype pattern creates objects by cloning existing ones. JavaScript's Object.create() implements this natively. Use for configuration defaults, template objects, and when you need copies with shared behavior.

## Code Examples

**Prototype pattern using Object.create() — clones inherit methods from the prototype chain, unlike structuredClone() which drops functions**

```javascript
const vehiclePrototype = {
  type: 'car',
  wheels: 4,
  describe() {
    return `${this.type} with ${this.wheels} wheels`;
  }
};

// Clone via Object.create() — inherits methods from the prototype
const truck = Object.create(vehiclePrototype);
truck.type = 'truck';
truck.wheels = 6;

const motorcycle = Object.create(vehiclePrototype);
motorcycle.type = 'motorcycle';
motorcycle.wheels = 2;

console.log(truck.describe());      // "truck with 6 wheels"
console.log(motorcycle.describe()); // "motorcycle with 2 wheels"

// Note: structuredClone() copies data but drops methods.
// Use Object.create() or spread ({...proto}) to preserve methods.
```


## Resources

- [MDN: Object.create()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create) — Object.create() reference
- [Refactoring Guru: Prototype](https://refactoring.guru/design-patterns/prototype) — Prototype pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*