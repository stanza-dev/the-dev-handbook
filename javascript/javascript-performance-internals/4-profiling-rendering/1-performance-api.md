---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-performance-api"
---

# Performance API

## Introduction

The Performance API provides precise timing measurements for profiling your code.

## Deep Dive

### High-Resolution Timing

```javascript
const start = performance.now();
doExpensiveWork();
const end = performance.now();
console.log(`Took ${end - start}ms`);
```

### Marks & Measures

```javascript
performance.mark('start-process');
await processData();
performance.mark('end-process');

performance.measure('processing', 'start-process', 'end-process');

const measures = performance.getEntriesByType('measure');
console.log(measures[0].duration);
```

### Resource Timing

```javascript
const resources = performance.getEntriesByType('resource');
resources.forEach(r => {
  console.log(`${r.name}: ${r.duration}ms`);
});
```

## Summary

performance.now() for precise timing. Marks and measures for custom profiling. Resource timing for network analysis.

## Resources

- [MDN: Performance API](https://developer.mozilla.org/en-US/docs/Web/API/Performance_API) — Performance API reference

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*