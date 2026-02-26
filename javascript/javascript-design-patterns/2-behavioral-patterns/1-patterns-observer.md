---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-observer"
---

# The Observer Pattern

## Introduction

The Observer pattern defines a one-to-many dependency between objects. When one object (subject) changes state, all dependents (observers) are notified. This is fundamental to event-driven programming.

## Key Concepts

**Subject/Observable**: Object being watched, maintains list of observers.

**Observer**: Object that wants to be notified of changes.

**Publish-Subscribe**: Variant with decoupled observers.

## Real World Context

Every modern UI framework is built on the Observer pattern. React's state updates, Vue's reactivity system, and RxJS observables all implement this pattern. The DOM's `addEventListener` is the most common Observer in web development. ES2025's `Set` methods like `union()`, `intersection()`, and `difference()` simplify managing subscriber sets:

```javascript
const newsSubscribers = new Set(['Alice', 'Bob', 'Carol']);
const sportsSubscribers = new Set(['Bob', 'Carol', 'Dave']);

// Subscribers in news but not sports
newsSubscribers.difference(sportsSubscribers); // Set {'Alice'}
// Subscribers in both channels
newsSubscribers.intersection(sportsSubscribers); // Set {'Bob','Carol'}
// All unique subscribers across both
newsSubscribers.union(sportsSubscribers); // Set {'Alice','Bob','Carol','Dave'}
```

## Deep Dive

### Basic Implementation

This event emitter stores callback arrays per event name. The `on()` method returns an unsubscribe function for easy cleanup:

```javascript
class EventEmitter {
  #listeners = new Map();
  
  on(event, callback) {
    if (!this.#listeners.has(event)) {
      this.#listeners.set(event, []);
    }
    this.#listeners.get(event).push(callback);
    return () => this.off(event, callback);
  }
  
  off(event, callback) {
    const callbacks = this.#listeners.get(event);
    if (callbacks) {
      const index = callbacks.indexOf(callback);
      if (index > -1) callbacks.splice(index, 1);
    }
  }
  
  emit(event, data) {
    const callbacks = this.#listeners.get(event);
    callbacks?.forEach(cb => cb(data));
  }
}

const emitter = new EventEmitter();
const unsubscribe = emitter.on('login', user => console.log(user));
emitter.emit('login', { name: 'Alice' });
unsubscribe();
```

The returned unsubscribe closure captures the `event` and `callback` references, making cleanup a one-liner for the consumer.

### Observable State

Combining Observer with state management creates a reactive store where any state change automatically notifies all subscribers:

```javascript
function createStore(initialState) {
  let state = initialState;
  const listeners = new Set();
  
  return {
    getState: () => state,
    setState(newState) {
      state = { ...state, ...newState };
      listeners.forEach(fn => fn(state));
    },
    subscribe(listener) {
      listeners.add(listener);
      return () => listeners.delete(listener);
    }
  };
}
```

Using a `Set` for listeners prevents duplicate subscriptions and makes removal O(1). The `subscribe` method returns an unsubscribe function following the same convention as the event emitter above.

## Common Pitfalls

1. **Memory leaks**: Forgetting to unsubscribe.
2. **Update storms**: Observers triggering each other.
3. **Callback hell**: Too many nested observers.

## Best Practices

1. **Always return an unsubscribe function** — Callers need a way to remove their listener to prevent memory leaks.
2. **Use `Set` for listeners** — ES2025 Set methods like `difference()` make managing subscriber lists cleaner and prevent duplicate subscriptions.
3. **Debounce high-frequency events** — Observers on scroll or resize events should be throttled to avoid performance bottlenecks.

## Summary

Observer pattern enables reactive programming. Subject notifies observers on changes. Always provide unsubscribe mechanism. DOM events and state libraries implement this pattern.

## Code Examples

**Observer pattern with Set-based listeners — on() returns an unsubscribe function to prevent memory leaks**

```javascript
class EventEmitter {
  #listeners = new Map();

  on(event, callback) {
    if (!this.#listeners.has(event)) {
      this.#listeners.set(event, new Set());
    }
    this.#listeners.get(event).add(callback);
    return () => this.#listeners.get(event).delete(callback);
  }

  emit(event, data) {
    this.#listeners.get(event)?.forEach(cb => cb(data));
  }
}

const emitter = new EventEmitter();
const unsub = emitter.on('login', user => console.log(`Welcome ${user}`));
emitter.emit('login', 'Alice'); // "Welcome Alice"
unsub(); // Removes listener
```


## Resources

- [Patterns.dev: Observer Pattern](https://www.patterns.dev/posts/observer-pattern/) — Observer pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*