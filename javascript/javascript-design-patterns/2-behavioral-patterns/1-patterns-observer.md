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

## Deep Dive

### Basic Implementation

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

### Observable State

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

## Common Pitfalls

1. **Memory leaks**: Forgetting to unsubscribe.
2. **Update storms**: Observers triggering each other.
3. **Callback hell**: Too many nested observers.

## Summary

Observer pattern enables reactive programming. Subject notifies observers on changes. Always provide unsubscribe mechanism. DOM events and state libraries implement this pattern.

## Resources

- [Patterns.dev: Observer Pattern](https://www.patterns.dev/posts/observer-pattern/) — Observer pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*