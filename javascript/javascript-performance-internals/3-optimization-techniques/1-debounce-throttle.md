---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-debounce-throttle"
---

## Introduction

Browsers fire certain events at an extremely high rate. A scroll event can trigger dozens of times per second, a resize event fires continuously while the user drags the window edge, and an input event fires on every single keystroke. If each event invocation triggers expensive work — DOM manipulation, network requests, or complex calculations — your application will grind to a halt.

**Debounce** and **Throttle** are two foundational rate-limiting techniques that every JavaScript developer must understand. They control how often a function executes in response to rapid events, and choosing the right one depends entirely on the use case.

## Key Concepts

- **Debounce**: Delays execution until a specified period of inactivity has passed. Each new invocation resets the timer. The function only fires once the rapid calls stop.
- **Throttle**: Ensures a function executes at most once within a fixed time window, regardless of how many times it is invoked.
- **Leading Edge**: Executing the function immediately on the first call, then suppressing subsequent calls for the duration of the delay.
- **Trailing Edge**: Executing the function after the delay period has elapsed since the last invocation. This is the default behavior for most debounce implementations.
- **Rate Limiting**: The general concept of controlling how frequently a function can be invoked, of which debounce and throttle are specific strategies.

## Real World Context

Debounce is the standard pattern for search-as-you-type: you do not want to fire an API request on every keystroke, but rather wait until the user pauses typing. Throttle is ideal for scroll-based animations or infinite scroll loading, where you need periodic updates but cannot afford to process every single scroll event. Google Maps throttles map tile loading during panning. Twitter debounces the character count API call in the compose box. Both patterns are critical for mobile performance where CPU and battery are constrained.

## Deep Dive

A debounce function works by maintaining a timer reference. Each time the debounced function is called, the previous timer is cleared and a new one is set. The actual function only executes when the timer completes without interruption:

```javascript
function debounce(fn, delay) {
  let timer;
  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}
```

A throttle function works differently. It allows the first call through immediately, then blocks subsequent calls until the time window expires:

```javascript
function throttle(fn, limit) {
  let inThrottle;
  return function(...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}
```

Usage for common events:

```javascript
// Search input — debounce so we wait for the user to stop typing
const searchInput = document.querySelector('#search');
searchInput.addEventListener('input', debounce(e => {
  fetchSearchResults(e.target.value);
}, 300));

// Scroll handler — throttle so we update at a controlled rate
window.addEventListener('scroll', throttle(() => {
  updateScrollProgress();
}, 100));

// Window resize — debounce since we only care about the final size
window.addEventListener('resize', debounce(() => {
  recalculateLayout();
}, 250));
```

For more advanced use cases, you can combine leading and trailing edge behavior. A leading-edge debounce fires immediately on the first call, then debounces subsequent calls. This is useful for button clicks where you want instant feedback but want to prevent double-clicks.

## Common Pitfalls

- **Losing `this` context**: Arrow functions inside debounce/throttle capture the wrong `this`. Using `fn.apply(this, args)` preserves the calling context correctly.
- **Not cleaning up timers**: When a component unmounts (in React or similar), failing to cancel pending debounced/throttled calls can cause state updates on unmounted components or memory leaks.
- **Choosing the wrong technique**: Using debounce for scroll animations causes visible jank because the handler only fires after scrolling stops. Use throttle for continuous feedback and debounce for final-value scenarios.

## Best Practices

- Use debounce for API calls triggered by user input (search, autocomplete, form validation) where you only need the final value.
- Use throttle for visual updates during continuous interactions (scroll position, resize, drag) where periodic updates keep the UI responsive.
- Always store the debounced/throttled function reference so you can cancel it during cleanup. In React, wrap the creation in `useMemo` or `useRef` to prevent re-creation on every render.

## Summary

Debounce and throttle are essential rate-limiting patterns for high-frequency events. Debounce waits for inactivity before firing, making it ideal for search inputs and resize handlers. Throttle fires at regular intervals during activity, making it perfect for scroll handlers and animations. Both preserve function context when implemented correctly and should always be cleaned up when components unmount.

## Code Examples

**Debounce and throttle implementations with usage examples**

```javascript
function debounce(fn, delay) {
  let timer;
  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

function throttle(fn, limit) {
  let inThrottle;
  return function(...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

// Usage
const handleSearch = debounce(query => fetchResults(query), 300);
const handleScroll = throttle(() => updatePosition(), 16);
```


## Resources

- [MDN: Debounce and Throttle](https://developer.mozilla.org/en-US/docs/Web/API/Document/scroll_event#scroll_event_throttling) — Event throttling patterns for scroll events

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*