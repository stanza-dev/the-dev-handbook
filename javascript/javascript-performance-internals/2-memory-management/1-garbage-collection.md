---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-garbage-collection"
---

# Garbage Collection Basics

## Introduction

JavaScript automatically manages memory through garbage collection. Understanding how it works helps you avoid memory leaks and write efficient code.

## Key Concepts

**Reachability**: Objects accessible from roots (global, stack) are kept.

**Mark and Sweep**: GC marks reachable objects, sweeps unmarked.

**Generational GC**: Young objects collected more often than old.

## Deep Dive

### V8's Garbage Collector

```
Young Generation (Minor GC - frequent)
├── Nursery: New allocations
└── Intermediate: Survived one GC

Old Generation (Major GC - less frequent)
└── Objects that survived multiple GCs
```

### GC Pauses

```javascript
// Creating many objects = more GC work
function badPattern() {
  for (let i = 0; i < 100000; i++) {
    const temp = { x: i, y: i * 2 };  // Garbage!
  }
}

// Object pooling reduces GC pressure
const pool = [];
function getObject() {
  return pool.pop() || { x: 0, y: 0 };
}
function releaseObject(obj) {
  obj.x = 0;
  obj.y = 0;
  pool.push(obj);
}
```

## Summary

GC automatically frees unreachable memory. Young objects collected frequently, old objects less often. Reduce allocations in hot paths to minimize GC pauses.

## Resources

- [MDN: Memory Management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management) — Memory management guide

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*