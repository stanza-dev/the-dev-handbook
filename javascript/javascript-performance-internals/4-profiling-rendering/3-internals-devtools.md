---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-devtools"
---

# DevTools Profiling

## Introduction

Chrome DevTools provides powerful profiling tools to find bottlenecks.

## Deep Dive

### Performance Panel

```
1. Open DevTools → Performance
2. Click Record
3. Perform actions
4. Stop recording
5. Analyze flame chart

Colors:
- Yellow: JavaScript
- Purple: Rendering
- Green: Painting
```

### Flame Chart Reading

```
Wide bars = slow functions
Deep stacks = many nested calls

Look for:
- Long tasks (>50ms)
- Forced reflows
- Expensive recalcs
```

### Console Profiling

```javascript
console.time('operation');
doWork();
console.timeEnd('operation');

console.profile('myProfile');
doWork();
console.profileEnd('myProfile');
```

## Summary

Performance panel for detailed analysis. Look for long tasks. console.time for quick measurements.

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*