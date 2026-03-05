---
source_course: "javascript"
source_lesson: "javascript-event-handling"
---

# Event Handling

## Introduction

Events are the heartbeat of interactive web applications—clicks, key presses, form submissions, mouse movements. JavaScript's event system lets you respond to user actions and create rich, dynamic experiences.

## Key Concepts

**Event**: A signal that something happened (user action or browser behavior).

**Event Listener**: A function that runs when a specific event occurs.

**Event Object**: Contains information about the event (target, coordinates, keys pressed, etc.).

## Real World Context

Form validation on submit, dropdown menus on hover, infinite scroll on scroll, keyboard shortcuts, drag and drop—events power all interactive features.

## Deep Dive

### Adding Event Listeners

```javascript
const button = document.querySelector('button');

// addEventListener (preferred)
button.addEventListener('click', function(event) {
  console.log('Clicked!', event.target);
});

// With arrow function
button.addEventListener('click', (e) => {
  console.log('Clicked!', e.target);
});

// Named function (easier to remove)
function handleClick(e) {
  console.log('Clicked!');
}
button.addEventListener('click', handleClick);
button.removeEventListener('click', handleClick);
```

### The Event Object

```javascript
element.addEventListener('click', (e) => {
  e.target;        // Element that triggered the event
  e.currentTarget; // Element the listener is attached to
  e.type;          // 'click'
  e.timeStamp;     // When the event occurred
  
  // Mouse events
  e.clientX, e.clientY;  // Viewport coordinates
  e.pageX, e.pageY;      // Page coordinates
  e.button;              // Which mouse button
  
  // Keyboard events
  e.key;         // 'Enter', 'a', 'ArrowUp'
  e.code;        // 'Enter', 'KeyA', 'ArrowUp'
  e.altKey, e.ctrlKey, e.shiftKey, e.metaKey;
});
```

### Common Events

```javascript
// Mouse events
element.addEventListener('click', handler);
element.addEventListener('dblclick', handler);
element.addEventListener('mouseenter', handler);  // No bubbling
element.addEventListener('mouseleave', handler);  // No bubbling
element.addEventListener('mouseover', handler);   // Bubbles

// Keyboard events
element.addEventListener('keydown', handler);  // Key pressed
element.addEventListener('keyup', handler);    // Key released

// Form events
form.addEventListener('submit', handler);
input.addEventListener('input', handler);   // Value changes
input.addEventListener('change', handler);  // Value committed
input.addEventListener('focus', handler);
input.addEventListener('blur', handler);

// Window events
window.addEventListener('load', handler);     // Page fully loaded
window.addEventListener('DOMContentLoaded', handler); // DOM ready
window.addEventListener('scroll', handler);
window.addEventListener('resize', handler);
```

### Preventing Default Behavior

```javascript
// Stop form submission
form.addEventListener('submit', (e) => {
  e.preventDefault();  // Form won't submit
  // Do custom validation/submission
});

// Stop link navigation
link.addEventListener('click', (e) => {
  e.preventDefault();  // Won't navigate
  // Do something else
});
```

### Event Listener Options

```javascript
// Run only once
button.addEventListener('click', handler, { once: true });

// Passive listener (better scroll performance)
window.addEventListener('scroll', handler, { passive: true });

// Capture phase (runs before bubbling)
parent.addEventListener('click', handler, { capture: true });
```

## Common Pitfalls

1. **Anonymous functions can't be removed**: Use named functions if you need to remove listeners.
2. **Event handler `this`**: In regular functions, `this` is the element; in arrows, it's lexical.
3. **Memory leaks**: Remove listeners when elements are removed.

## Best Practices

- **Use `addEventListener`**: Not inline `onclick` attributes.
- **Use `passive` for scroll/touch**: Improves performance.
- **Remove listeners when done**: Prevents memory leaks.
- **Check `e.target` vs `e.currentTarget`**: They can be different.

## Summary

Use `addEventListener` to attach event handlers. The event object contains details about the event. Use `preventDefault()` to stop default browser behavior. Use options like `once` and `passive` for specific behaviors. Always clean up listeners to prevent memory leaks.

## Code Examples

**Adding Event Listeners**

```javascript
const button = document.querySelector('button');

// addEventListener (preferred)
button.addEventListener('click', function(event) {
  console.log('Clicked!', event.target);
});

// With arrow function
button.addEventListener('click', (e) => {
  console.log('Clicked!', e.target);
});

// Named function (easier to remove)
function handleClick(e) {
  console.log('Clicked!');
}
button.addEventListener('click', handleClick);
button.removeEventListener('click', handleClick);
```

**The Event Object**

```javascript
element.addEventListener('click', (e) => {
  e.target;        // Element that triggered the event
  e.currentTarget; // Element the listener is attached to
  e.type;          // 'click'
  e.timeStamp;     // When the event occurred
  
  // Mouse events
  e.clientX, e.clientY;  // Viewport coordinates
  e.pageX, e.pageY;      // Page coordinates
  e.button;              // Which mouse button
  
  // Keyboard events
  e.key;         // 'Enter', 'a', 'ArrowUp'
  e.code;        // 'Enter', 'KeyA', 'ArrowUp'
  e.altKey, e.ctrlKey, e.shiftKey, e.metaKey;
});
```


## Resources

- [MDN: EventTarget.addEventListener()](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener) — addEventListener reference
- [MDN: Event reference](https://developer.mozilla.org/en-US/docs/Web/Events) — Complete list of DOM events

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*