---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-dom-optimization"
---

## Introduction

The DOM is the most expensive API surface in browser JavaScript. Every time you read a layout property or write a style change, you potentially trigger the browser to recalculate styles, recompute layout, repaint pixels, and recomposite layers. A single misplaced DOM read inside a write loop can force the browser to perform synchronous layout hundreds of times in a single frame, a problem known as layout thrashing.

Understanding how the browser processes DOM changes and structuring your code to work with the rendering pipeline rather than against it is essential for building performant web applications.

## Key Concepts

- **DOM Reflow** (Layout): The browser recalculates the position and geometry of elements in the document. Triggered by changes to dimensions, position, font size, or reading layout properties like `offsetWidth` or `getBoundingClientRect()`.
- **Repaint**: The browser redraws the visual appearance of elements without changing their geometry. Triggered by changes to color, visibility, background, or box-shadow.
- **Layout Thrashing**: A performance antipattern where DOM reads and writes are interleaved in a loop, forcing the browser to perform synchronous layout on every iteration instead of batching changes.
- **DocumentFragment**: A lightweight DOM node that acts as a staging area for building DOM subtrees. Appending a fragment to the document triggers only a single reflow regardless of how many child nodes it contains.
- **Virtual DOM**: A programming concept where a lightweight JavaScript representation of the DOM is maintained in memory and diffed against previous versions to compute the minimal set of real DOM operations needed. Used by React and similar frameworks.

## Real World Context

Layout thrashing is one of the most common causes of jank in web applications. It frequently occurs in animation code, table rendering, and responsive layout calculations. Facebook's early news feed suffered from layout thrashing when calculating post heights for virtual scrolling. Gmail's compose window uses DocumentFragment to build complex email formatting without intermediate reflows. CSS containment is used extensively in large applications like Google Docs to isolate layout calculations to individual document sections.

## Deep Dive

The most critical optimization is separating DOM reads from DOM writes. When you read a layout property (like `offsetWidth`) after modifying styles, the browser must perform a synchronous layout to return an accurate value:

```javascript
// BAD: Layout thrashing — read + write in each iteration
for (const el of elements) {
  el.style.width = container.offsetWidth + 'px';
}

// GOOD: Batch reads, then batch writes
const width = container.offsetWidth; // Single read
for (const el of elements) {
  el.style.width = width + 'px'; // Writes only
}
```

When inserting many elements, use `DocumentFragment` to batch them into a single DOM operation:

```javascript
// BAD: 1000 individual insertions = 1000 potential reflows
for (let i = 0; i < 1000; i++) {
  const li = document.createElement('li');
  li.textContent = `Item ${i}`;
  list.appendChild(li);
}

// GOOD: Single insertion via fragment = 1 reflow
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const li = document.createElement('li');
  li.textContent = `Item ${i}`;
  fragment.appendChild(li);
}
list.appendChild(fragment);
```

For DOM updates that depend on the current rendering state, use `requestAnimationFrame` to schedule writes at the optimal time in the rendering cycle:

```javascript
requestAnimationFrame(() => {
  // Reads
  const rect = element.getBoundingClientRect();
  // Writes (in the same rAF callback, after all reads)
  element.style.transform = `translateY(${rect.top}px)`;
});
```

CSS containment tells the browser that an element's internals are independent from the rest of the page, allowing it to skip large portions of the rendering pipeline:

```css
.card {
  contain: layout style paint;
}

.offscreen-section {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px;
}
```

The `will-change` property hints to the browser that an element will be animated, allowing it to promote the element to its own compositor layer ahead of time:

```css
.animated {
  will-change: transform, opacity;
}
```

However, overusing `will-change` creates excessive layers that consume GPU memory. Apply it only to elements that will actually animate, and remove it after the animation completes.

## Common Pitfalls

- **Reading layout properties inside loops**: Properties like `offsetHeight`, `clientWidth`, `scrollTop`, and `getBoundingClientRect()` force synchronous layout when read after a write. Extract these reads before the loop.
- **Overusing will-change**: Promoting too many elements to their own compositor layers wastes GPU memory and can actually degrade performance. Only apply it to elements that will be animated.
- **Forgetting content-visibility**: For long pages with off-screen content, failing to use `content-visibility: auto` means the browser lays out and paints thousands of invisible elements.

## Best Practices

- Always batch DOM reads together, then batch DOM writes together. Never interleave reads and writes.
- Use `DocumentFragment` or `innerHTML` for bulk insertions rather than appending elements one at a time.
- Apply `contain: layout style paint` to independent UI components (cards, list items, widgets) to isolate their rendering cost from the rest of the page.

## Summary

DOM optimization centers on minimizing reflows and repaints by batching reads and writes, using DocumentFragment for bulk insertions, scheduling updates with requestAnimationFrame, and leveraging CSS containment to isolate rendering work. Layout thrashing is the single most impactful DOM performance problem, and the fix is always the same: read first, then write.

## Code Examples

**Avoiding layout thrashing and using DocumentFragment**

```javascript
// BAD: Layout thrashing
for (const el of elements) {
  el.style.width = container.offsetWidth + 'px'; // Read + write each iteration
}

// GOOD: Batch reads then writes
const width = container.offsetWidth; // Single read
for (const el of elements) {
  el.style.width = width + 'px'; // Writes only
}

// GOOD: DocumentFragment for batch inserts
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const li = document.createElement('li');
  li.textContent = `Item ${i}`;
  fragment.appendChild(li);
}
list.appendChild(fragment); // Single reflow
```


## Resources

- [MDN: DocumentFragment](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment) — Using DocumentFragment for efficient DOM manipulation
- [MDN: CSS Containment](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment) — CSS containment for rendering performance

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*