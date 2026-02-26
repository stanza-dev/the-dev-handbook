---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-weak-references"
---

## Introduction

JavaScript's garbage collector cannot reclaim an object as long as any strong reference points to it. But sometimes you want to reference an object without preventing its collection, for example, in a cache where entries should disappear when memory is tight. This is where **weak references** come in.

ES2021 introduced `WeakRef` and `FinalizationRegistry` as low-level primitives for advanced memory management patterns. Combined with the existing `WeakMap` and `WeakSet`, these tools give you fine-grained control over object lifecycle and cleanup.

## Key Concepts

- **WeakRef**: A wrapper that holds a weak reference to an object. You can call `.deref()` to get the object if it is still alive, or `undefined` if it has been garbage collected. Unlike strong references, a WeakRef does not prevent GC.

- **FinalizationRegistry**: A mechanism to register cleanup callbacks that run after an object is garbage collected. Useful for releasing external resources (file handles, network connections) associated with GC'd objects.

- **WeakMap**: A key-value collection where keys must be objects and are held weakly. If nothing else references a key, the entry is automatically removed. Ideal for associating metadata with objects without preventing their collection.

- **WeakSet**: A set of objects held weakly. Useful for tracking whether an object has been processed or seen without preventing its collection.

- **Weak vs Strong Reference**: A strong reference (normal variable, property, array element) keeps an object alive. A weak reference allows the GC to collect the object when no strong references remain. This distinction is fundamental to preventing memory leaks in caches and registries.

## Real World Context

Weak references are essential for building memory-efficient caches, memoization layers, and observer patterns. React's internal fiber tree uses WeakMap-like patterns to associate metadata with component instances. Build tools and bundlers use WeakMap to cache transform results keyed by AST nodes.

In server-side applications, WeakRef-based caches allow the GC to automatically evict entries under memory pressure, providing a natural back-pressure mechanism without explicit TTL management.

## Deep Dive

**WeakRef for memory-safe caching:**

```javascript
class WeakCache {
  #cache = new Map();
  #registry = new FinalizationRegistry(key => {
    // Called after the cached value is GC'd
    this.#cache.delete(key);
    console.log(`Cache entry '${key}' was garbage collected`);
  });

  set(key, value) {
    const ref = new WeakRef(value);
    this.#cache.set(key, ref);
    this.#registry.register(value, key);
  }

  get(key) {
    const ref = this.#cache.get(key);
    if (!ref) return undefined;
    const value = ref.deref();
    if (!value) {
      // Object was GC'd but FinalizationRegistry hasn't fired yet
      this.#cache.delete(key);
      return undefined;
    }
    return value;
  }
}
```

**WeakMap for private data:**

```javascript
const privateData = new WeakMap();

class User {
  constructor(name, ssn) {
    this.name = name;
    // SSN is stored in WeakMap, not on the object itself
    privateData.set(this, { ssn });
  }

  getSSN() {
    return privateData.get(this).ssn;
  }
}

const user = new User('Alice', '123-45-6789');
console.log(user.name);    // 'Alice'
console.log(user.getSSN()); // '123-45-6789'
// When 'user' is GC'd, the WeakMap entry is automatically removed
```

**WeakSet for deduplication:**

```javascript
const processed = new WeakSet();

function processOnce(obj) {
  if (processed.has(obj)) return; // Already processed
  processed.add(obj);
  // ... expensive processing ...
}
```

**ES2025 Explicit Resource Management (`using` / `await using`):**

As a complement to weak references for non-deterministic cleanup, TC39 has standardized explicit resource management with the `using` keyword and `Symbol.dispose` / `Symbol.asyncDispose` protocols. While `FinalizationRegistry` callbacks run at an unpredictable time after GC, `using` provides deterministic cleanup when a scope exits:

```javascript
// Deterministic cleanup with Symbol.dispose
class DatabaseConnection {
  [Symbol.dispose]() {
    this.close();
    console.log('Connection closed deterministically');
  }
}

function queryData() {
  using conn = new DatabaseConnection();
  // ... use conn ...
  // conn[Symbol.dispose]() is called automatically when scope exits
}

// Async version with Symbol.asyncDispose
class FileHandle {
  async [Symbol.asyncDispose]() {
    await this.flush();
    await this.close();
  }
}

async function processFile() {
  await using file = new FileHandle('data.txt');
  // ... use file ...
  // file[Symbol.asyncDispose]() called when scope exits
}
```

Use `FinalizationRegistry` as a safety net for resources that might not be explicitly disposed, and `using`/`await using` for deterministic cleanup in known scopes.

## Common Pitfalls

- **Relying on FinalizationRegistry for critical cleanup**: GC timing is non-deterministic. Cleanup callbacks may run much later than expected, or not at all if the process exits. Never use FinalizationRegistry as your only cleanup mechanism for critical resources.

- **Calling deref() and caching the result**: If you store the result of `weakRef.deref()` in a variable, you have created a strong reference. Only hold the dereferenced value for the duration you need it.

- **Using WeakMap/WeakSet with primitive keys**: WeakMap and WeakSet only accept objects as keys. Strings, numbers, and other primitives cannot be used because they are not garbage collected in the same way.

## Best Practices

- **Use WeakMap for metadata association**: Whenever you need to attach extra data to objects you don't own (DOM nodes, third-party objects), use WeakMap instead of expando properties to avoid leaks.

- **Combine WeakRef with FinalizationRegistry**: Use WeakRef to hold the reference and FinalizationRegistry to clean up associated entries in regular Maps or other data structures when the object is collected.

- **Prefer `using`/`await using` for deterministic cleanup**: When you control the lifecycle scope, use explicit resource management instead of relying on GC. Reserve FinalizationRegistry as a fallback safety net.

## Summary

Weak references allow you to reference objects without preventing garbage collection. WeakRef and FinalizationRegistry provide low-level control for caches and resource cleanup. WeakMap and WeakSet offer higher-level weak-keyed collections for metadata association and membership tracking. The new ES2025 `using`/`await using` syntax complements these tools by providing deterministic resource cleanup at scope boundaries. Together, these primitives enable memory-efficient patterns that cooperate with the garbage collector.

## Code Examples

**WeakRef and FinalizationRegistry for memory-safe caching**

```javascript
// WeakRef-based cache that doesn't prevent GC
class WeakCache {
  #cache = new Map();
  #registry = new FinalizationRegistry(key => {
    this.#cache.delete(key);
  });

  set(key, value) {
    const ref = new WeakRef(value);
    this.#cache.set(key, ref);
    this.#registry.register(value, key);
  }

  get(key) {
    const ref = this.#cache.get(key);
    return ref?.deref(); // undefined if GC'd
  }
}
```


## Resources

- [MDN: WeakRef](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakRef) — WeakRef API for weak object references
- [MDN: FinalizationRegistry](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/FinalizationRegistry) — FinalizationRegistry for cleanup callbacks

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*