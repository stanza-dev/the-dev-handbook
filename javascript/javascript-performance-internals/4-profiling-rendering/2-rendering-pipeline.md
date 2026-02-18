---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-rendering-pipeline"
---

# Browser Rendering Pipeline

## Introduction

Understanding the rendering pipeline helps you write code that doesn't cause unnecessary work.

## Deep Dive

### The Pipeline

```
JavaScript → Style → Layout → Paint → Composite
```

### What Triggers What

```javascript
// Triggers Layout, Paint, Composite
element.style.width = '100px';

// Triggers Paint, Composite only
element.style.color = 'red';

// Triggers Composite only (cheapest!)
element.style.transform = 'translateX(100px)';
element.style.opacity = 0.5;
```

### Layout Thrashing

```javascript
// Bad: Forces synchronous layout
elements.forEach(el => {
  el.style.width = el.offsetWidth + 10 + 'px';
});

// Good: Batch reads, then writes
const widths = elements.map(el => el.offsetWidth);
elements.forEach((el, i) => {
  el.style.width = widths[i] + 10 + 'px';
});
```

## Summary

Transform and opacity are cheapest. Geometric changes trigger layout. Batch reads before writes.

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*