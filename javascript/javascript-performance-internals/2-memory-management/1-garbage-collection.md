---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-garbage-collection"
---

## Introduction

JavaScript is a garbage-collected language, meaning the engine automatically reclaims memory that is no longer in use. While this frees developers from manual memory management, it introduces its own set of performance considerations. Garbage collection (GC) pauses can cause frame drops, jank, and latency spikes if you are not mindful of how and when objects are allocated.

V8 uses a sophisticated garbage collector called **Orinoco** that employs generational collection, incremental marking, and concurrent sweeping to minimize pause times. Understanding how Orinoco works helps you write code that cooperates with the GC rather than fighting against it.

## Key Concepts

- **Mark-and-Sweep**: The fundamental GC algorithm. The collector "marks" all objects reachable from root references (global scope, stack, etc.), then "sweeps" (frees) unmarked objects. Any object not reachable from a root is considered garbage.

- **Generational GC**: V8 divides the heap into two main spaces based on object age. The hypothesis is that most objects die young, so separating young and old objects allows more efficient collection strategies for each.

- **Young Generation/Scavenger**: A small memory region (typically 1-8 MB) where newly allocated objects live. Collected frequently using a fast "Scavenger" algorithm (a semi-space copying collector). Most objects are collected here without ever reaching old generation.

- **Old Generation/Major GC**: Where objects that survive multiple young generation collections are promoted. Collected less frequently using Mark-Sweep-Compact, which is more expensive but handles long-lived objects efficiently.

- **Incremental Marking**: Instead of stopping the world to mark all reachable objects at once, V8 interleaves small chunks of marking work with JavaScript execution. This spreads GC pauses across multiple small increments rather than one large pause.

## Real World Context

GC pauses are one of the primary causes of animation jank in web applications. A major GC pause of 50-100ms during a scroll or animation is clearly visible to users. Game engines written in JavaScript (like Phaser or Three.js) must carefully manage object allocation to avoid GC pauses during gameplay.

In Node.js servers, GC pauses can cause request latency spikes. High-throughput services may need to tune V8's heap size (`--max-old-space-size`) and monitor GC activity to maintain consistent response times.

## Deep Dive

V8's Orinoco garbage collector uses a generational approach with two main spaces:

**Young Generation (Scavenger):**

The young generation uses a semi-space design: memory is split into two equal halves called "from-space" and "to-space". New objects are allocated in from-space. When from-space fills up:

1. Live objects in from-space are copied to to-space.
2. The roles of from-space and to-space are swapped.
3. Objects that survive two scavenge cycles are promoted to old generation.

```javascript
function processData(items) {
  // These temporary objects are allocated in young generation
  const mapped = items.map(item => ({ ...item, processed: true }));
  const filtered = mapped.filter(item => item.active);
  // 'mapped' and spread copies become garbage after this function returns
  // Collected cheaply by Scavenger
  return filtered.length;
}
```

**Old Generation (Major GC):**

Objects promoted from young generation live in old generation. Major GC uses Mark-Sweep-Compact:

1. **Mark**: Traverse the object graph from roots, marking reachable objects.
2. **Sweep**: Free memory occupied by unmarked objects.
3. **Compact**: Optionally move surviving objects together to reduce fragmentation.

```javascript
const cache = new Map(); // Allocated in young gen, promoted to old gen

function getCached(key) {
  if (!cache.has(key)) {
    cache.set(key, expensiveCompute(key));
  }
  return cache.get(key);
}
// cache lives for the entire application lifetime → old generation
```

**Incremental and Concurrent Collection:**

Modern V8 performs marking concurrently on background threads, with the main thread only pausing briefly for finalization. This reduces pause times from hundreds of milliseconds to single-digit milliseconds:

```
Main Thread:  [JS][pause][JS][pause][JS][final pause][JS]
BG Thread:    [----marking----|----marking----]
```

## Common Pitfalls

- **Creating excessive short-lived objects in hot loops**: Each allocation increases GC pressure. Allocating thousands of small objects per frame in an animation loop triggers frequent scavenge pauses.

- **Ignoring GC in latency-sensitive code**: If your application has strict latency requirements (real-time audio, animation, server response times), you must account for GC pauses in your performance budget.

- **Growing the old generation unboundedly**: Long-lived data structures that grow without bounds (caches without eviction, accumulating event logs) force increasingly expensive major GC cycles.

## Best Practices

- **Reduce allocation rate in hot paths**: Reuse objects and arrays where possible. Object pooling can dramatically reduce GC pressure in performance-critical code like game loops or real-time data processing.

- **Prefer short-lived objects**: Design your code so most objects are temporary and collected cheaply by the Scavenger. Avoid promoting objects to old generation unnecessarily by keeping their lifetime short.

- **Monitor GC with performance tooling**: Use Chrome DevTools Performance tab (look for GC events), Node.js `--trace-gc` flag, or `performance.measureUserAgentSpecificMemory()` to understand your application's GC behavior.

## Summary

V8's Orinoco garbage collector uses generational collection to efficiently manage memory. Short-lived objects are collected cheaply in the young generation by the Scavenger. Long-lived objects are promoted to old generation and collected by the more expensive Mark-Sweep-Compact algorithm. Incremental and concurrent marking minimize GC pause times. Writing GC-friendly code means reducing allocation rate in hot paths, preferring short-lived objects, and monitoring GC behavior in production.

## Code Examples

**Young vs old generation object lifecycles**

```javascript
// Short-lived objects stay in Young Generation
function processData(items) {
  // These temporary objects are collected by Scavenger (minor GC)
  const mapped = items.map(item => ({ ...item, processed: true }));
  const filtered = mapped.filter(item => item.active);
  return filtered.length;
}

// Long-lived objects promote to Old Generation
const cache = new Map(); // Survives multiple GC cycles
function getCached(key) {
  if (!cache.has(key)) {
    cache.set(key, expensiveCompute(key));
  }
  return cache.get(key);
}
```


## Resources

- [V8 Blog: Trash Talk - Orinoco GC](https://v8.dev/blog/trash-talk) — Deep dive into V8's Orinoco garbage collector

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*