---
source_course: "react-animation"
source_lesson: "react-animation-performance-principles"
---

# Animation Performance Principles

Achieving smooth 60fps animations requires understanding browser rendering.

## The Rendering Pipeline

1. **JavaScript** - Calculate animations
2. **Style** - Compute styles
3. **Layout** - Calculate positions and sizes
4. **Paint** - Fill in pixels
5. **Composite** - Layer compositing

## Compositor-Only Properties

These skip layout and paint:

```css
/* ✅ Fast - compositor only */
.element {
  transform: translateX(100px);
  transform: scale(1.5);
  transform: rotate(45deg);
  opacity: 0.5;
}

/* ❌ Slow - triggers layout */
.element {
  width: 200px;
  height: 200px;
  top: 100px;
  left: 100px;
  margin: 20px;
  padding: 10px;
}

/* ⚠️ Medium - triggers paint */
.element {
  background-color: red;
  border-color: blue;
  box-shadow: 0 0 10px black;
}
```

## will-change Property

Hint to browser for optimization:

```css
/* Use sparingly! */
.animated-element {
  will-change: transform, opacity;
}

/* Remove when not animating */
.animated-element.idle {
  will-change: auto;
}
```

```jsx
// In React, apply dynamically
function AnimatedCard({ isAnimating }) {
  return (
    <div
      style={{
        willChange: isAnimating ? 'transform' : 'auto',
        transform: isAnimating ? 'scale(1.1)' : 'scale(1)',
      }}
    >
      Content
    </div>
  );
}
```

## GPU Acceleration

```css
/* Force GPU layer */
.accelerated {
  transform: translateZ(0);
  /* or */
  transform: translate3d(0, 0, 0);
}
```

## Avoid Layout Thrashing

```jsx
// ❌ Bad: Reading and writing alternately
items.forEach((item) => {
  const height = item.offsetHeight; // Read
  item.style.height = height + 10 + 'px'; // Write
});

// ✅ Good: Batch reads, then batch writes
const heights = items.map((item) => item.offsetHeight);
items.forEach((item, i) => {
  item.style.height = heights[i] + 10 + 'px';
});
```

## React-Specific Tips

```jsx
// Use CSS instead of inline styles for static values
// ❌ Recreates object every render
<div style={{ transform: 'translateX(0)' }} />

// ✅ CSS class
<div className="initial-position" />

// ✅ Or memoize
const style = useMemo(() => ({ transform: 'translateX(0)' }), []);
<div style={style} />
```

## Measuring Performance

```js
// Check for dropped frames
let lastTime = performance.now();
function checkFPS() {
  const now = performance.now();
  const delta = now - lastTime;
  if (delta > 16.67) { // More than 1 frame at 60fps
    console.log(`Dropped frame: ${delta.toFixed(2)}ms`);
  }
  lastTime = now;
  requestAnimationFrame(checkFPS);
}
```

---

> 📘 *This lesson is part of the [React Animation & Motion](https://stanza.dev/courses/react-animation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*