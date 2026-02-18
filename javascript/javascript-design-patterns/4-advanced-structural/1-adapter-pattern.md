---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-adapter-pattern"
---

# The Adapter Pattern

## Introduction

The Adapter pattern allows incompatible interfaces to work together by wrapping one interface to match another expected by the client.

## Deep Dive

### Interface Adapter

```javascript
// Legacy API
class LegacyCalculator {
  compute(op, a, b) {
    if (op === 'add') return a + b;
    if (op === 'sub') return a - b;
  }
}

// Adapter to new interface
class CalculatorAdapter {
  #calc = new LegacyCalculator();
  
  add(a, b) { return this.#calc.compute('add', a, b); }
  subtract(a, b) { return this.#calc.compute('sub', a, b); }
}
```

### Storage Adapter

```javascript
const localStorageAdapter = {
  get(key) {
    const item = localStorage.getItem(key);
    return item ? JSON.parse(item) : null;
  },
  set(key, value) {
    localStorage.setItem(key, JSON.stringify(value));
  }
};

const memoryAdapter = {
  #store: new Map(),
  get(key) { return this.#store.get(key); },
  set(key, value) { this.#store.set(key, value); }
};
```

## Summary

Adapters wrap incompatible interfaces. Use for legacy integration, API versioning, and swappable implementations.

## Resources

- [Refactoring Guru: Adapter](https://refactoring.guru/design-patterns/adapter) — Adapter pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*