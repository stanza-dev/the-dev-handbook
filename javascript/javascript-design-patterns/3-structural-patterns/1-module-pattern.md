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

The IIFE creates a closure that keeps `count` and `validateCount` private. Only the returned object is accessible from outside:

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

Accessing `CounterModule.count` returns `undefined` because `count` is a closure variable, not a property of the returned object. The only way to read it is through `getCount()`.

### Revealing Module Pattern

The Revealing Module variation defines all functions inside the closure and then returns an object that maps public names to those private functions:

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

The returned object has short-hand property names that point to the private functions. This makes the public API explicit and easy to scan at a glance.

### ES2025 Import Attributes

Import attributes (ES2025) let you specify the expected module type, preventing security issues where a server might serve unexpected content:

```javascript
// Import JSON with type assertion (ES2025)
import config from './config.json' with { type: 'json' };
import styles from './theme.css' with { type: 'css' };

// Dynamic import with attributes
const data = await import('./data.json', { with: { type: 'json' } });
```

The `with { type: 'json' }` syntax works with both static and dynamic imports. Import attributes tell the engine how to interpret a module, preventing security issues where a server might serve unexpected content types.

### Modern ES Modules

ES modules provide built-in encapsulation. Variables declared at module scope are private by default, and only explicitly `export`ed members are accessible:

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

The `count` variable is module-scoped and cannot be accessed by importers. This is the modern equivalent of the IIFE module pattern, with static analysis and tree-shaking support.

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

## Code Examples

**ES module pattern with private state — import attributes (ES2025) enable typed imports like JSON with { type: 'json' }**

```javascript
// ES module with import attributes (ES2025)
import config from './config.json' with { type: 'json' };

// Private to module scope
let count = 0;

export function increment() {
  return ++count;
}

export function getCount() {
  return count;
}

// Usage in another file:
// import { increment, getCount } from './counter.js';
// increment(); // 1
// count; // ReferenceError — private to module
```


## Resources

- [MDN: JavaScript modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) — ES modules guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*