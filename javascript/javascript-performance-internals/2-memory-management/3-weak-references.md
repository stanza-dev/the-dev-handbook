---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-weak-references"
---

# WeakMap, WeakSet & WeakRef

## Introduction

Weak collections hold 'weak' references that don't prevent garbage collection. Use them for caches and metadata that shouldn't keep objects alive.

## Deep Dive

### WeakMap

```javascript
// Map keeps keys alive
const map = new Map();
let obj = { data: 'large' };
map.set(obj, 'metadata');
obj = null;  // Object still exists in map!

// WeakMap allows collection
const weakMap = new WeakMap();
let obj2 = { data: 'large' };
weakMap.set(obj2, 'metadata');
obj2 = null;  // Object can be collected!
```

### Private Data Pattern

```javascript
const privateData = new WeakMap();

class User {
  constructor(name, password) {
    privateData.set(this, { password });
    this.name = name;
  }
  
  checkPassword(pwd) {
    return privateData.get(this).password === pwd;
  }
}
// When User instance is collected, private data is too
```

### WeakRef (ES2021)

```javascript
const cache = new Map();

function getCached(key, compute) {
  let ref = cache.get(key);
  if (ref) {
    const value = ref.deref();
    if (value !== undefined) return value;
  }
  const value = compute(key);
  cache.set(key, new WeakRef(value));
  return value;
}
```

## Summary

WeakMap/WeakSet don't prevent GC of keys. Use for caches, private data, DOM metadata. WeakRef provides even weaker references. Object may disappear at any time.

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*