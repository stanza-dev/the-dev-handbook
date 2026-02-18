---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-debounce-throttle"
---

# Debounce & Throttle

## Introduction

High-frequency events like scroll, resize, and input can fire hundreds of times per second. Debouncing and throttling limit how often handlers run, preventing performance issues.

## Key Concepts

**Debounce**: Wait for pause in events before executing.

**Throttle**: Execute at most once per time period.

**Leading/Trailing**: Execute at start or end of interval.

## Deep Dive

### Debounce Implementation

```javascript
function debounce(fn, delay) {
  let timeout;
  return function(...args) {
    clearTimeout(timeout);
    timeout = setTimeout(() => fn.apply(this, args), delay);
  };
}

// Usage: Search after user stops typing
const search = debounce((query) => {
  fetch(`/search?q=${query}`);
}, 300);

input.addEventListener('input', e => search(e.target.value));
```

### Throttle Implementation

```javascript
function throttle(fn, interval) {
  let lastTime = 0;
  return function(...args) {
    const now = Date.now();
    if (now - lastTime >= interval) {
      lastTime = now;
      fn.apply(this, args);
    }
  };
}

// Usage: Scroll handler at 60fps
const onScroll = throttle(() => {
  updateScrollPosition();
}, 16);
```

### When to Use Each

- **Debounce**: Search input, form validation, window resize
- **Throttle**: Scroll events, mouse move, game loops

## Common Pitfalls

1. **Too long delay**: UI feels unresponsive.
2. **Too short delay**: Doesn't help performance.
3. **Memory leaks**: Not clearing timeouts on unmount.

## Summary

Debounce waits for pause. Throttle limits rate. Choose based on whether you need final value (debounce) or regular updates (throttle).

## Resources

- [MDN: Performance](https://developer.mozilla.org/en-US/docs/Web/Performance) — Web performance guide

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*