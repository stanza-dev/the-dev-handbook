---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-rendering-pipeline"
---

## Introduction

Every visual change on a web page passes through a multi-stage rendering pipeline before pixels appear on screen. Understanding this pipeline — and knowing which CSS properties trigger which stages — is the key to eliminating unnecessary rendering work. An animation that triggers only compositing runs at 60fps with zero main-thread involvement, while one that triggers layout can cause visible jank even on powerful hardware.

This lesson explains each stage of the rendering pipeline, identifies which CSS properties trigger which stages, and teaches you to choose properties that minimize rendering cost.

## Key Concepts

- **Rendering Pipeline**: The sequence of steps the browser performs to turn your HTML, CSS, and JavaScript into pixels: JavaScript -> Style -> Layout -> Paint -> Composite.
- **Style Calculation**: The browser determines which CSS rules apply to each DOM element by matching selectors, resolving specificity, computing inheritance, and producing the final computed style for every element.
- **Layout (Reflow)**: Using the computed styles, the browser calculates the exact position and dimensions of every element in the document. Changes to width, height, margin, padding, position, or font-size trigger layout.
- **Paint**: The browser fills in pixels for each element — drawing text, colors, images, borders, and shadows. This happens in software onto paint records that describe the drawing operations.
- **Composite**: The browser takes independently painted layers and combines them into the final screen image. This stage is GPU-accelerated and handles properties like `transform` and `opacity` without involving the main thread.
- **Layer Promotion**: The browser promotes certain elements to their own compositor layer (e.g., elements with `will-change: transform` or `transform: translateZ(0)`), allowing them to be composited independently for better animation performance.

## Real World Context

CSS animation libraries like Framer Motion and GSAP default to animating `transform` and `opacity` because these are compositor-only properties that skip layout and paint entirely. Chrome's Layers panel in DevTools shows which elements have been promoted to their own layer and why. Google's Core Web Vitals metric Cumulative Layout Shift (CLS) directly measures unexpected layout operations, penalizing pages that trigger frequent reflows. Mobile browsers are especially sensitive to rendering performance because their CPUs are slower and battery life is limited.

## Deep Dive

The rendering pipeline runs in a specific order, and each stage can be skipped if the change does not require it:

**Full pipeline (JavaScript -> Style -> Layout -> Paint -> Composite):**
Triggered by properties that affect geometry: `width`, `height`, `margin`, `padding`, `top`, `left`, `font-size`, `display`, `position`, `float`.

**Skip layout (JavaScript -> Style -> Paint -> Composite):**
Triggered by properties that affect appearance but not geometry: `color`, `background-color`, `box-shadow`, `border-color`, `visibility`, `outline`.

**Skip layout and paint (JavaScript -> Style -> Composite):**
Triggered by compositor-only properties: `transform`, `opacity`, `filter`, `backdrop-filter`. These can be handled entirely on the GPU without touching the main thread.

```javascript
// Compositor-only — fast path
element.style.transform = 'translateX(100px)'; // Composite only
element.style.opacity = 0.5;                   // Composite only

// Triggers paint (skips layout)
element.style.backgroundColor = 'red';          // Style + Paint + Composite
element.style.color = 'blue';                   // Style + Paint + Composite

// Triggers full pipeline
element.style.width = '200px';                  // Style + Layout + Paint + Composite
element.style.marginLeft = '10px';              // Style + Layout + Paint + Composite
```

To promote an element to its own compositor layer for smoother animation:

```css
.animated-element {
  will-change: transform;
}

/* Alternative (older technique) */
.animated-element {
  transform: translateZ(0);
}
```

The browser also performs style recalculation whenever the DOM changes or classes are toggled. The cost of style calculation grows with the number of elements and the complexity of your CSS selectors. Deeply nested selectors like `.sidebar > .nav > ul > li > a.active` are more expensive to match than flat selectors like `.nav-link--active`.

For animations, always prefer animating `transform` and `opacity`. Instead of animating `top` or `left` (which trigger layout on every frame), use `transform: translate()`:

```css
/* BAD: Triggers layout every frame */
@keyframes slide-bad {
  from { left: 0; }
  to { left: 300px; }
}

/* GOOD: Compositor-only, GPU-accelerated */
@keyframes slide-good {
  from { transform: translateX(0); }
  to { transform: translateX(300px); }
}
```

## Common Pitfalls

- **Animating layout properties**: Animating `width`, `height`, `top`, `left`, `margin`, or `padding` forces layout recalculation on every frame, causing jank. Use `transform` and `opacity` for all animations.
- **Excessive layer promotion**: Using `will-change` on too many elements wastes GPU memory and can cause compositing to become the bottleneck instead of layout. Only promote elements that will actually animate.
- **Complex CSS selectors**: Deeply nested, universal, or attribute selectors increase the cost of style calculation. Keep selectors flat and specific.

## Best Practices

- Animate only `transform` and `opacity` for smooth 60fps animations that skip layout and paint entirely.
- Use the Chrome DevTools Rendering tab to enable "Paint flashing" and "Layout shift regions" to visualize which elements are being repainted or reflowed.
- Apply `will-change` sparingly and only just before an animation starts. Remove it after the animation completes to free GPU memory.

## Summary

The browser rendering pipeline runs in five stages: JavaScript, Style, Layout, Paint, and Composite. Each CSS property change triggers a specific subset of these stages. Compositor-only properties (`transform`, `opacity`) skip layout and paint for GPU-accelerated rendering. Understanding which properties trigger which pipeline stages allows you to choose the fastest path for every visual change, resulting in smooth 60fps animations and responsive interactions.

## Code Examples

**Compositor-only vs layout-triggering CSS changes**

```javascript
// Compositor-only animation (no layout/paint)
element.style.transform = 'translateX(100px)'; // Fast
element.style.opacity = 0.5; // Fast

// Triggers layout + paint (slow)
element.style.width = '200px'; // Layout
element.style.backgroundColor = 'red'; // Paint

// Promote to own layer for better animation
.animated-element {
  will-change: transform;
  /* or: transform: translateZ(0); */
}
```


## Resources

- [Chrome: Rendering Performance](https://web.dev/articles/rendering-performance) — Understanding the browser rendering pipeline

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*