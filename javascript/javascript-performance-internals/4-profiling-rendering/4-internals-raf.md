---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-raf"
---

# requestAnimationFrame

## Introduction

rAF synchronizes JavaScript with the browser's rendering cycle for smooth animations.

## Deep Dive

### Basic Usage

```javascript
function animate(timestamp) {
  element.style.transform = `translateX(${x}px)`;
  x += 2;
  if (x < 300) requestAnimationFrame(animate);
}
requestAnimationFrame(animate);
```

### Why Not setInterval?

```javascript
// Bad: May drift, runs when hidden
setInterval(animate, 16);

// Good: Syncs with display, pauses when hidden
requestAnimationFrame(animate);
```

### Frame-Rate Independent

```javascript
let lastTime = 0;
function animate(timestamp) {
  const dt = timestamp - lastTime;
  lastTime = timestamp;
  x += speed * dt;  // Consistent regardless of FPS
  requestAnimationFrame(animate);
}
```

## Summary

rAF syncs with display refresh. Pauses when tab hidden. Use deltaTime for consistent animation speed.

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*