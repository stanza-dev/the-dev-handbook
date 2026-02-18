---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-memory-profiling"
---

# Memory Profiling Tools

## Introduction

Chrome DevTools provides powerful memory profiling to find leaks and understand allocation patterns.

## Deep Dive

### Heap Snapshot

```
DevTools → Memory → Take heap snapshot

Views:
- Summary: Objects grouped by constructor
- Comparison: Diff between snapshots
- Containment: Object reference tree
- Dominators: Objects keeping others alive
```

### Allocation Timeline

```
DevTools → Memory → Allocation instrumentation

- Shows allocations over time
- Blue bars = surviving objects
- Gray bars = collected objects
- Investigate blue bars that grow over time
```

### Finding Leaks

```javascript
// 1. Take initial snapshot
// 2. Perform suspected leak action
// 3. Force GC (trash icon)
// 4. Take second snapshot
// 5. Compare snapshots
// 6. Look for unexpected retained objects

// In code:
console.memory;  // Heap size info
performance.memory;  // More detailed (Chrome)
```

### Allocation Sampling

```
DevTools → Memory → Allocation sampling

- Lower overhead than timeline
- Good for production profiling
- Shows allocation call stacks
```

## Summary

Use heap snapshots to find what's retained. Allocation timeline shows growth over time. Compare snapshots to find leaks. Force GC before snapshots for accuracy.

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*