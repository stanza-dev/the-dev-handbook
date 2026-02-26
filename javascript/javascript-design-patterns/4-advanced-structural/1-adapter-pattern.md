---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-adapter-pattern"
---

# The Adapter Pattern

## Introduction

The Adapter pattern allows incompatible interfaces to work together by wrapping one interface to match another expected by the client.

## Key Concepts

**Adapter**: A wrapper that translates one interface into another.

**Adaptee**: The existing class with an incompatible interface.

**Target Interface**: The interface the client expects.

## Real World Context

API version migration, third-party library integration, and legacy system wrapping all use adapters. When you switch from one HTTP client to another (e.g., Axios to Fetch), you write an adapter so the rest of your code doesn't change.

## Deep Dive

### Interface Adapter

This adapter wraps a legacy calculator with a single `compute(op, a, b)` method and exposes a modern interface with named methods:

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

Client code calls `adapter.add(2, 3)` instead of `legacy.compute('add', 2, 3)`. The legacy class remains unchanged and could be swapped out transparently.

### Storage Adapter

Storage adapters let you swap between localStorage, memory, or any other backend while keeping the same `get`/`set` interface:

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

Both adapters implement the same `get`/`set` interface. Swapping `localStorageAdapter` for `memoryAdapter` in your application requires no other code changes.

## Common Pitfalls

1. **Adapter chains** — Adapting an adapter creates layers of indirection. Limit to one level.
2. **Leaking the adaptee** — Exposing methods from the underlying library defeats the purpose of adaptation.
3. **Incomplete adaptation** — Wrapping only some methods leaves consumers with a half-usable interface.

## Best Practices

1. **Define the target interface first** — Write the interface your code expects, then build the adapter to match.
2. **Use adapters at integration boundaries** — Wrap third-party libraries at import time so internal code never sees the raw API.
3. **Write adapters as thin wrappers** — An adapter should translate, not add business logic.

## Summary

Adapters wrap incompatible interfaces. Use for legacy integration, API versioning, and swappable implementations.

## Code Examples

**Adapter wrapping a legacy calculator — the client uses the clean modern interface while the adapter translates to the old API**

```javascript
// Legacy API with different interface
class LegacyCalc {
  compute(op, a, b) {
    if (op === 'add') return a + b;
    if (op === 'sub') return a - b;
  }
}

// Adapter: maps new interface → legacy
class CalcAdapter {
  #legacy = new LegacyCalc();
  add(a, b)      { return this.#legacy.compute('add', a, b); }
  subtract(a, b) { return this.#legacy.compute('sub', a, b); }
}

const calc = new CalcAdapter();
calc.add(5, 3);      // 8
calc.subtract(5, 3); // 2
```


## Resources

- [Refactoring Guru: Adapter](https://refactoring.guru/design-patterns/adapter) — Adapter pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*