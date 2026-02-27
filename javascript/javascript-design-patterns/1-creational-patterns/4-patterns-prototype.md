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

## Deep Dive

### Object.create()

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

### Clone with Customization

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

### Deep Clone Pattern

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
    if (obj.hasOwnProperty(key)) {
      cloned[key] = deepClone(obj[key]);
    }
  }
  return cloned;
}

// Modern: structuredClone()
const clone = structuredClone(original);
```

### Registry of Prototypes

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

## Resources

- [MDN: Object.create()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create) — Object.create() reference
- [Refactoring Guru: Prototype](https://refactoring.guru/design-patterns/prototype) — Prototype pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*