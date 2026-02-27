---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-dom-optimization"
---

# DOM Optimization

## Introduction

DOM operations are expensive. Minimizing access and batching changes dramatically improves performance.

## Deep Dive

### Batch Reads and Writes

```javascript
// Bad: Causes layout thrashing
elements.forEach(el => {
  const height = el.offsetHeight;  // Read
  el.style.height = height + 10 + 'px';  // Write
});

// Good: Batch all reads, then writes
const heights = elements.map(el => el.offsetHeight);
elements.forEach((el, i) => {
  el.style.height = heights[i] + 10 + 'px';
});
```

### DocumentFragment

```javascript
// Bad: Many reflows
for (let i = 0; i < 1000; i++) {
  ul.appendChild(document.createElement('li'));
}

// Good: Single reflow
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  fragment.appendChild(document.createElement('li'));
}
ul.appendChild(fragment);
```

### CSS Transforms

```javascript
// Bad: Triggers layout
element.style.left = x + 'px';

// Good: GPU-accelerated, no layout
element.style.transform = `translateX(${x}px)`;
```

## Summary

Batch DOM reads before writes. Use DocumentFragment for bulk inserts. Prefer transform/opacity for animations.

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*