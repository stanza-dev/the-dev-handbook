---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-raf"
---

## Introduction

Before `requestAnimationFrame` (rAF), JavaScript animations relied on `setInterval` or `setTimeout` with hardcoded delays, leading to inconsistent frame rates, wasted CPU cycles when the tab was hidden, and animations that drifted out of sync with the display's refresh rate. rAF solved all of these problems by letting the browser schedule your animation callback at exactly the right moment — just before the next repaint.

Mastering rAF means understanding its callback timing, using the high-resolution timestamp parameter for frame-independent animation, building efficient animation loops, and leveraging advanced techniques like the double-rAF trick for post-paint timing.

## Key Concepts

- **requestAnimationFrame**: A browser API that schedules a callback to run just before the next repaint. The browser automatically syncs the callback to the display's refresh rate (typically 60Hz or 120Hz), pauses it when the tab is not visible, and batches callbacks for efficiency.
- **cancelAnimationFrame**: Cancels a previously scheduled rAF callback using the ID returned by `requestAnimationFrame`. Essential for cleanup when animations are interrupted or components unmount.
- **Frame Budget**: At 60fps, each frame has 16.67ms for all work: JavaScript execution, style calculation, layout, paint, and compositing. If your JavaScript takes more than about 10ms, the frame will be dropped because the remaining pipeline stages also need time.
- **Jank**: Visible stuttering caused by dropped frames. When a frame takes longer than the frame budget, the browser cannot update the display at the expected rate, producing a noticeable hitch.
- **DOMHighResTimeStamp**: The high-resolution timestamp (in milliseconds with microsecond precision) passed to the rAF callback. It represents the time at the beginning of the current frame and is consistent across all rAF callbacks in the same frame.

## Real World Context

Every modern animation library uses rAF internally. GSAP, Framer Motion, Anime.js, and even CSS-in-JS animation solutions schedule their updates through rAF. Game engines running in the browser (Phaser, Three.js) use rAF as their core game loop. Infinite scroll implementations use rAF-based throttling for scroll handlers. The `IntersectionObserver` and `ResizeObserver` APIs internally schedule their callbacks relative to the rendering pipeline in a similar manner.

## Deep Dive

The rAF callback receives a `DOMHighResTimeStamp` representing when the current batch of rAF callbacks was triggered. Use this timestamp for frame-independent animation rather than calculating elapsed time yourself:

```javascript
let startTime;
function animate(timestamp) {
  if (!startTime) startTime = timestamp;
  const elapsed = timestamp - startTime;
  const progress = Math.min(elapsed / 1000, 1); // 1 second duration
  
  element.style.transform = `translateX(${progress * 300}px)`;
  
  if (progress < 1) {
    requestAnimationFrame(animate);
  }
}
requestAnimationFrame(animate);
```

This animation runs for exactly 1 second regardless of frame rate. On a 60Hz display it takes 60 frames, on a 120Hz display it takes 120 frames, but the total duration is the same because we use elapsed time, not frame count.

rAF-based throttling is the preferred pattern for scroll and resize handlers because it naturally aligns with the display's rendering cycle:

```javascript
let ticking = false;
window.addEventListener('scroll', () => {
  if (!ticking) {
    requestAnimationFrame(() => {
      updateScrollPosition();
      ticking = false;
    });
    ticking = true;
  }
});
```

This ensures `updateScrollPosition` runs at most once per frame, no matter how many scroll events fire between frames.

The double-rAF trick schedules code to run after the browser has painted the current frame. A single rAF runs before paint; a nested rAF inside it runs before the next paint, meaning the code between them runs after the first paint:

```javascript
requestAnimationFrame(() => {
  // This runs before paint
  element.classList.add('visible');
  
  requestAnimationFrame(() => {
    // This runs after the previous paint has completed
    // Useful for measuring the rendered result
    const rect = element.getBoundingClientRect();
    console.log('Painted at:', rect);
  });
});
```

This technique is useful for reading layout after a visual change has been committed, or for triggering CSS transitions that require the initial state to be painted first.

Comparing rAF with `setInterval` for animation:

```javascript
// BAD: setInterval does not sync with display refresh
setInterval(() => {
  element.style.left = (pos++) + 'px';
}, 16); // Approximately 60fps, but drifts

// GOOD: rAF syncs perfectly with display refresh
function animate() {
  element.style.transform = `translateX(${pos++}px)`;
  requestAnimationFrame(animate);
}
requestAnimationFrame(animate);
```

`setInterval` fires based on a timer that is independent of the display's vsync. It can fire in the middle of a frame (wasting the update), skip frames under load, or accumulate drift over time. rAF fires once per frame at exactly the right time, is automatically paused in background tabs, and provides a precise timestamp.

## Common Pitfalls

- **Not canceling rAF on cleanup**: In single-page applications, failing to call `cancelAnimationFrame` when a component unmounts causes the animation loop to continue running, referencing stale DOM elements and leaking memory.
- **Using setInterval for visual updates**: `setInterval` cannot sync with the display's refresh rate, wastes CPU in background tabs (rAF pauses automatically), and produces inconsistent frame timing. Always use rAF for anything visual.
- **Assuming 60fps**: Not all displays run at 60Hz. High-refresh-rate monitors (120Hz, 144Hz) call rAF more frequently. Always use the elapsed time from the timestamp parameter for animation math, never assume a fixed frame rate.

## Best Practices

- Always use the `timestamp` parameter passed to the rAF callback for elapsed time calculations. This ensures animations run at the correct speed regardless of frame rate.
- Store the rAF ID returned by `requestAnimationFrame` and cancel it with `cancelAnimationFrame` during cleanup (React `useEffect` return, component unmount, animation interruption).
- Keep your rAF callback under 10ms to leave room for the browser's rendering pipeline within the 16.67ms frame budget at 60fps.

## Summary

requestAnimationFrame is the foundation of smooth browser animation. It syncs with the display's refresh rate, pauses in background tabs, and provides a high-resolution timestamp for frame-independent animation. Use rAF-based throttling for scroll and resize handlers to limit updates to one per frame. The double-rAF trick enables post-paint timing. Always cancel rAF handles on cleanup and use the timestamp parameter for elapsed time calculations rather than assuming any particular frame rate.

## Code Examples

**Animation loop and rAF-based scroll throttling**

```javascript
// Smooth animation loop
let startTime;
function animate(timestamp) {
  if (!startTime) startTime = timestamp;
  const elapsed = timestamp - startTime;
  const progress = Math.min(elapsed / 1000, 1); // 1s duration
  
  element.style.transform = `translateX(${progress * 300}px)`;
  
  if (progress < 1) {
    requestAnimationFrame(animate);
  }
}
requestAnimationFrame(animate);

// rAF-based throttle
let ticking = false;
window.addEventListener('scroll', () => {
  if (!ticking) {
    requestAnimationFrame(() => {
      updateScrollPosition();
      ticking = false;
    });
    ticking = true;
  }
});
```


## Resources

- [MDN: requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame) — requestAnimationFrame for smooth animations

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*