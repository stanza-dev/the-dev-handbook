---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-memory-profiling"
---

## Introduction

Knowing about garbage collection and memory leaks is only half the battle. The other half is being able to diagnose memory issues in real applications. **Memory profiling** gives you the tools to measure heap usage, identify leaked objects, and understand what is keeping memory alive. Chrome DevTools and Node.js provide powerful profiling capabilities that every JavaScript developer should know how to use.

Memory problems are notoriously difficult to reproduce and diagnose because they are often gradual. A heap snapshot taken at any single moment may look fine; it is the comparison between snapshots over time that reveals the leak.

## Key Concepts

- **Heap Snapshot**: A complete record of all objects on the JavaScript heap at a point in time. It includes object types, sizes, and reference relationships. Comparing two snapshots reveals which objects were allocated between them.

- **Allocation Timeline**: A recording mode that tracks every object allocation over time. Each allocation is shown on a timeline, making it easy to correlate allocations with specific user actions or code paths.

- **Retained Size vs Shallow Size**: Shallow size is the memory directly held by an object (its own fields). Retained size is the total memory that would be freed if the object were garbage collected, including all objects it exclusively references.

- **Dominator Tree**: A tree structure showing which objects "dominate" (keep alive) other objects. Object A dominates object B if every path from GC roots to B passes through A. The dominator tree helps identify which object is responsible for retaining a large subtree of memory.

- **Memory Pressure**: When the application approaches the heap limit, V8 runs more frequent and aggressive GC cycles. This manifests as increased GC pauses, higher CPU usage, and degraded application responsiveness.

## Real World Context

Memory profiling is a critical skill for production debugging. When users report that your web app becomes sluggish after extended use, or your Node.js server's memory grows until it crashes, memory profiling is how you identify the root cause.

Companies like Netflix, Airbnb, and LinkedIn have published detailed case studies about finding and fixing memory leaks in production JavaScript applications. The common thread is always the same: take heap snapshots, compare them, and follow the retainer chain to find the leak.

## Deep Dive

**Taking heap snapshots in Chrome DevTools:**

1. Open DevTools and go to the Memory tab.
2. Select "Heap snapshot" and click "Take snapshot".
3. Perform the actions that you suspect cause a leak.
4. Take another snapshot.
5. Switch to "Comparison" view between snapshot 1 and snapshot 2.
6. Sort by "# New" or "Size Delta" to find objects that were allocated but not freed.

**The three-snapshot technique:**

The gold standard for finding leaks:

1. Take Snapshot 1 (baseline).
2. Perform the suspect action (e.g., open and close a modal).
3. Take Snapshot 2.
4. Perform the same action again.
5. Take Snapshot 3.
6. Compare Snapshot 3 to Snapshot 1. Objects present in Snapshot 3 but not in Snapshot 1, and that should have been cleaned up, are leaks.

Doing the action twice eliminates one-time allocations (like lazy initialization) from the results.

**Finding retainers:**

When you find a suspicious object in a heap snapshot, click on it to see its "Retainers" panel. This shows the chain of references keeping the object alive. Follow the chain upward to find the root cause, which is often an event listener, closure, or global variable.

**Programmatic memory measurement:**

```javascript
// Legacy API (Chrome only, approximate)
if (performance.memory) {
  console.log('Used JS Heap:', performance.memory.usedJSHeapSize);
  console.log('Total JS Heap:', performance.memory.totalJSHeapSize);
  console.log('Heap Limit:', performance.memory.jsHeapSizeLimit);
}

// Modern API (requires cross-origin isolation)
async function measureMemory() {
  if (typeof performance.measureUserAgentSpecificMemory === 'function') {
    const result = await performance.measureUserAgentSpecificMemory();
    console.log('Total bytes:', result.bytes);
    for (const entry of result.breakdown) {
      console.log(`  ${entry.types.join(', ')}: ${entry.bytes} bytes`);
      for (const attr of entry.attribution) {
        console.log(`    from: ${attr.url}`);
      }
    }
  }
}
```

**Node.js memory profiling:**

```javascript
// Built-in process.memoryUsage()
const usage = process.memoryUsage();
console.log({
  rss: `${(usage.rss / 1024 / 1024).toFixed(1)} MB`,       // Resident Set Size
  heapTotal: `${(usage.heapTotal / 1024 / 1024).toFixed(1)} MB`,
  heapUsed: `${(usage.heapUsed / 1024 / 1024).toFixed(1)} MB`,
  external: `${(usage.external / 1024 / 1024).toFixed(1)} MB`,
});

// Generate heap snapshot from code
const v8 = require('v8');
const fs = require('fs');
const snapshotStream = v8.writeHeapSnapshot();
console.log('Heap snapshot written to:', snapshotStream);
// Load the .heapsnapshot file in Chrome DevTools Memory tab
```

**Using `--trace-gc` in Node.js:**

```bash
node --trace-gc server.js
# Output:
# [GC] Scavenge 2.3 (3.0) -> 1.8 (4.0) MB, 1.2 / 0.0 ms
# [GC] Mark-sweep 45.1 (50.0) -> 30.2 (50.0) MB, 25.3 / 0.0 ms
```

This shows every GC event with heap sizes before and after collection, helping you understand allocation rates and GC efficiency.

## Common Pitfalls

- **Taking only one snapshot**: A single snapshot tells you what is in memory now, but not what should or should not be there. Always compare snapshots to identify unexpected growth.

- **Ignoring retained size**: An object with 100 bytes of shallow size might retain 100 MB of other objects through its references. Always sort by retained size when hunting leaks.

- **Profiling in development mode**: Framework development modes often add extra debugging data, ref tracking, and error overlays that consume significant memory. Always profile production builds for accurate results.

## Best Practices

- **Use the three-snapshot technique**: Baseline, action, repeat action, compare. This reliably separates leaks from one-time allocations and framework overhead.

- **Automate memory checks in CI**: Use `process.memoryUsage()` in integration tests to detect memory regression. Run a workload, trigger GC with `global.gc()` (requires `--expose-gc`), and assert heap usage is within bounds.

- **Set up memory monitoring in production**: Use APM tools (Datadog, New Relic, or custom metrics) to track heap usage over time. Alert on monotonically increasing memory that correlates with traffic but not with request count.

## Summary

Memory profiling is the practical skill that turns theoretical knowledge of GC and leaks into actionable debugging. Heap snapshots reveal what objects exist and what retains them. The allocation timeline shows when objects are created. The three-snapshot comparison technique reliably identifies leaks. Programmatic APIs and Node.js tools enable automated memory monitoring. Master these tools and you can diagnose any memory issue in a JavaScript application.

## Code Examples

**Programmatic memory measurement APIs**

```javascript
// Programmatic memory measurement
console.log('Heap used:', performance.memory?.usedJSHeapSize);

// Mark heap for snapshot comparison
console.log('Taking snapshot 1...');
// ... perform operations ...
console.log('Taking snapshot 2...');

// Using performance.measureUserAgentSpecificMemory() (modern)
async function checkMemory() {
  if (crossOriginIsolated) {
    const result = await performance.measureUserAgentSpecificMemory();
    console.log('Total bytes:', result.bytes);
    result.breakdown.forEach(entry => {
      console.log(entry.types, entry.bytes);
    });
  }
}
```


## Resources

- [Chrome DevTools: Memory](https://developer.chrome.com/docs/devtools/memory) — Chrome DevTools memory profiling tools

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*