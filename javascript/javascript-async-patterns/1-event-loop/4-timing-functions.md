---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-timing-functions"
---

# Timers and Scheduling

## Introduction

JavaScript provides several ways to schedule code execution: setTimeout, setInterval, requestAnimationFrame, and requestIdleCallback. Each serves different purposes—understanding when to use which leads to better performance and smoother UIs.

## Key Concepts

**setTimeout**: Execute once after delay.

**setInterval**: Execute repeatedly at intervals.

**requestAnimationFrame**: Execute before next paint (60fps).

**requestIdleCallback**: Execute when browser is idle.

## Real World Context

Debouncing user input, animating smoothly, polling for updates, deferring non-critical work—scheduling is fundamental to performant web applications.

## Deep Dive

### setTimeout

```javascript
// Basic usage
const timeoutId = setTimeout(() => {
  console.log('Executed after 1 second');
}, 1000);

// Cancel before execution
clearTimeout(timeoutId);

// With arguments
setTimeout((a, b) => console.log(a + b), 1000, 5, 3);  // 8

// Minimum delay is ~4ms (nested timeouts)
// 0ms doesn't mean immediate!
```

### setInterval

```javascript
let count = 0;
const intervalId = setInterval(() => {
  count++;
  console.log('Tick:', count);
  if (count >= 5) {
    clearInterval(intervalId);  // Stop after 5
  }
}, 1000);

// Problem: drift over time
// If callback takes 50ms, interval is really 1050ms

// Better: self-correcting timer
function preciseInterval(callback, interval) {
  let expected = Date.now() + interval;
  
  function step() {
    const drift = Date.now() - expected;
    callback();
    expected += interval;
    setTimeout(step, Math.max(0, interval - drift));
  }
  
  setTimeout(step, interval);
}
```

### requestAnimationFrame

```javascript
// For animations - synced to display refresh (usually 60fps)
function animate(element) {
  let position = 0;
  
  function step() {
    position += 2;
    element.style.transform = `translateX(${position}px)`;
    
    if (position < 300) {
      requestAnimationFrame(step);  // Schedule next frame
    }
  }
  
  requestAnimationFrame(step);
}

// Timestamp parameter
requestAnimationFrame((timestamp) => {
  console.log('Time since page load:', timestamp);
});

// Cancel
const frameId = requestAnimationFrame(callback);
cancelAnimationFrame(frameId);
```

### requestIdleCallback

```javascript
// Run when browser is idle (low priority)
requestIdleCallback((deadline) => {
  // deadline.timeRemaining() - ms until browser needs control
  // deadline.didTimeout - true if timeout was reached
  
  while (deadline.timeRemaining() > 0 && tasks.length) {
    processTask(tasks.shift());
  }
  
  if (tasks.length) {
    requestIdleCallback(processRemainingTasks);  // Continue later
  }
}, { timeout: 2000 });  // Max wait time

// Use for: analytics, prefetching, non-critical updates
```

### Comparison

```javascript
// setTimeout: General delays
// - Debouncing, delayed actions, timeouts

// setInterval: Repeating tasks
// - Polling, clocks (but beware drift)

// requestAnimationFrame: Visual updates
// - Animations, scroll effects, canvas
// - Pauses when tab is hidden (saves CPU)

// requestIdleCallback: Low priority
// - Analytics, prefetch, background work
// - Not supported in Safari (use polyfill)
```

## Common Pitfalls

1. **setInterval drift**: Callbacks take time; use self-correcting timers.
2. **Memory leaks**: Forgetting to clear intervals on component unmount.
3. **Blocking timers**: Long sync code delays scheduled callbacks.

## Best Practices

- **Use requestAnimationFrame for animations**: Smooth 60fps, auto-pauses.
- **Clear timers on cleanup**: Prevent leaks in SPAs.
- **Use requestIdleCallback for analytics**: Don't block critical work.
- **Consider drift in intervals**: Self-correct for precision.

## Summary

setTimeout runs once after delay; setInterval repeats. requestAnimationFrame syncs with display refresh for smooth animations. requestIdleCallback runs during idle time for non-critical work. Always clear timers to prevent memory leaks.

## Code Examples

**setTimeout**

```javascript
// Basic usage
const timeoutId = setTimeout(() => {
  console.log('Executed after 1 second');
}, 1000);

// Cancel before execution
clearTimeout(timeoutId);

// With arguments
setTimeout((a, b) => console.log(a + b), 1000, 5, 3);  // 8

// Minimum delay is ~4ms (nested timeouts)
// 0ms doesn't mean immediate!
```

**setInterval**

```javascript
let count = 0;
const intervalId = setInterval(() => {
  count++;
  console.log('Tick:', count);
  if (count >= 5) {
    clearInterval(intervalId);  // Stop after 5
  }
}, 1000);

// Problem: drift over time
// If callback takes 50ms, interval is really 1050ms

// Better: self-correcting timer
function preciseInterval(callback, interval) {
  let expected = Date.now() + interval;
  
  function step() {
    const drift = Date.now() - expected;
```


## Resources

- [MDN: setTimeout()](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout) — setTimeout reference
- [MDN: requestAnimationFrame()](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) — requestAnimationFrame reference

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*