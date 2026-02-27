---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-module-pattern"
---

# The Module Pattern

## Introduction

The Module pattern encapsulates code into self-contained units with public and private members. Before ES6 modules, this was the primary way to avoid polluting the global namespace and create private state.

## Key Concepts

**IIFE**: Immediately Invoked Function Expression creating a closure.

**Public Interface**: Returned object exposing public methods.

**Private Members**: Variables/functions inside the closure, inaccessible from outside.

## Real World Context

While ES modules are preferred today, the Module pattern still appears in legacy code, browser scripts without module bundlers, and understanding closures.

## Deep Dive

### Classic Module Pattern

```javascript
const CounterModule = (function() {
  // Private members
  let count = 0;
  
  function validateCount(n) {
    return typeof n === 'number' && n >= 0;
  }
  
  // Public interface
  return {
    increment() {
      count++;
      return count;
    },
    decrement() {
      if (count > 0) count--;
      return count;
    },
    getCount() {
      return count;
    },
    reset() {
      count = 0;
    }
  };
})();

CounterModule.increment();  // 1
CounterModule.count;        // undefined (private!)
```

### Revealing Module Pattern

```javascript
const Calculator = (function() {
  let result = 0;
  
  function add(n) {
    result += n;
    return result;
  }
  
  function subtract(n) {
    result -= n;
    return result;
  }
  
  function getResult() {
    return result;
  }
  
  function reset() {
    result = 0;
  }
  
  // Reveal public pointers to private functions
  return {
    add,
    subtract,
    getResult,
    reset
  };
})();
```

### Modern ES Modules

```javascript
// counter.js
let count = 0;  // Private to module

export function increment() {
  return ++count;
}

export function getCount() {
  return count;
}

// main.js
import { increment, getCount } from './counter.js';
increment();  // 1
```

## Common Pitfalls

1. **Memory leaks**: Closures hold references to outer scope.
2. **Testing difficulties**: Can't mock private members.
3. **No dynamic exports**: Can't add to public interface after creation.

## Best Practices

- **Prefer ES modules**: Native support, static analysis.
- **Use for browser scripts**: When modules aren't available.
- **Keep modules focused**: Single responsibility.
- **Document the public interface**: Clear API.

## Summary

The Module pattern uses closures to create private state. ES modules are the modern replacement with native `import`/`export`. Use the pattern when modules aren't available or for understanding closure-based privacy.

## Resources

- [MDN: JavaScript modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) — ES modules guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*