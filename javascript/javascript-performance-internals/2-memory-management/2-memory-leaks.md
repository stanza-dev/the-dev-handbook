---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-memory-leaks"
---

## Introduction

A **memory leak** occurs when your application retains references to objects that are no longer needed, preventing the garbage collector from reclaiming that memory. Over time, leaks cause your application to consume more and more memory, leading to degraded performance, increased GC pressure, and eventually crashes.

Memory leaks in JavaScript are particularly insidious because the language is garbage-collected. Developers often assume the GC handles everything, but it can only collect objects that are truly unreachable. If your code accidentally holds a reference to an object, the GC considers it alive, no matter how much memory it consumes.

## Key Concepts

- **Memory Leak**: A situation where memory that is no longer needed cannot be reclaimed because references to it still exist somewhere in the application. The memory grows monotonically over time.

- **Detached DOM Tree**: A DOM subtree that has been removed from the document but is still referenced by JavaScript. Common in SPAs where components are dynamically created and destroyed.

- **Closure Leak**: When a closure captures variables from its outer scope that reference large objects, and the closure itself is retained longer than necessary (e.g., as an event handler or callback).

- **Forgotten Timer**: A `setInterval` or `setTimeout` callback that references objects but is never cleared. The timer keeps the callback alive, which keeps the referenced objects alive.

- **Event Listener Leak**: Event listeners registered on DOM elements or EventEmitters that are never removed. Each listener holds a reference to its callback closure and potentially to the target element.

## Real World Context

Memory leaks are the number one cause of "my app gets slower over time" complaints. SPAs are especially susceptible because they run for extended periods without full page reloads. A small leak that accumulates 1 KB per navigation can consume hundreds of megabytes in a long-running session.

In Node.js, memory leaks can bring down production servers. Without the browser's page-reload safety valve, a leaking Node.js process will eventually exceed its memory limit and be killed by the OS or container orchestrator.

## Deep Dive

Let us examine the five most common memory leak patterns:

**1. Closure retaining large data:**

```javascript
function createHandler() {
  const largeData = new Array(1000000).fill('x');
  return function onClick() {
    // largeData is captured in this closure's scope
    // Even though onClick never uses largeData,
    // some engines may retain it (depends on scope analysis)
    console.log('clicked');
  };
}
document.getElementById('btn').addEventListener('click', createHandler());
```

Fix: Nullify large references after use, or restructure to avoid capturing them.

```javascript
function createHandler() {
  let largeData = new Array(1000000).fill('x');
  const result = processData(largeData);
  largeData = null; // Allow GC to reclaim
  return function onClick() {
    console.log('result:', result);
  };
}
```

**2. Forgotten setInterval:**

```javascript
function startPolling(element) {
  setInterval(() => {
    // This closure references 'element'
    element.textContent = fetchStatus();
  }, 1000);
  // If 'element' is removed from DOM, the interval still runs
  // and holds a reference to the detached element
}
```

Fix: Store the interval ID and clear it when the component unmounts.

**3. Detached DOM trees:**

```javascript
let detachedTree;
function createWidget() {
  const widget = document.createElement('div');
  widget.innerHTML = '<span>Large widget content...</span>';
  document.body.appendChild(widget);
  detachedTree = widget; // Global reference
}
function removeWidget() {
  document.body.removeChild(detachedTree);
  // detachedTree still references the removed DOM tree!
  // Fix: detachedTree = null;
}
```

**4. Accumulating global state:**

```javascript
const logs = [];
function logEvent(event) {
  logs.push({ ...event, timestamp: Date.now() });
  // logs array grows forever!
  // Fix: limit size or use a ring buffer
}
```

**5. Event listeners not cleaned up:**

```javascript
class Component {
  mount() {
    this.handler = () => this.update();
    window.addEventListener('resize', this.handler);
  }
  unmount() {
    // LEAK: forgot to remove listener!
    // Fix: window.removeEventListener('resize', this.handler);
  }
}
```

## Common Pitfalls

- **Using anonymous functions as event listeners**: You cannot remove an anonymous function listener because you have no reference to it. Always store a reference to the handler function.

- **Not cleaning up in framework lifecycle hooks**: React's `useEffect` cleanup, Angular's `ngOnDestroy`, and Vue's `onUnmounted` exist specifically to clean up listeners, timers, and subscriptions.

- **Relying on the GC to handle everything**: The garbage collector only collects unreachable objects. If your code maintains a reference path to an object, the GC considers it intentionally alive.

## Best Practices

- **Always pair setup with teardown**: For every `addEventListener`, have a corresponding `removeEventListener`. For every `setInterval`, have a `clearInterval`. For every subscription, have an unsubscribe.

- **Use AbortController for fetch and listeners**: `AbortController` provides a clean way to cancel fetch requests and remove multiple event listeners in one call.

- **Take heap snapshots regularly**: In Chrome DevTools, take a heap snapshot, perform operations, take another snapshot, and compare. Objects that only appear in the second snapshot and should not be there are likely leaks.

## Summary

Memory leaks in JavaScript happen when references to unused objects prevent garbage collection. The five most common leak patterns are closure leaks, forgotten timers, detached DOM trees, accumulating global state, and unremoved event listeners. Prevention requires disciplined cleanup in lifecycle hooks, using weak references where appropriate, and regular memory profiling with heap snapshots.

## Code Examples

**Closure-based memory leak and fix pattern**

```javascript
// LEAK: Closure retains outer scope
function createLeak() {
  const largeData = new Array(1000000).fill('x');
  return function() {
    // largeData is retained even if never accessed
    console.log('handler called');
  };
}

// FIX: Nullify reference when done
function createFixed() {
  let largeData = new Array(1000000).fill('x');
  const result = processData(largeData);
  largeData = null; // Allow GC
  return function() {
    console.log('result:', result);
  };
}
```


## Resources

- [MDN: Memory Management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_management) — JavaScript memory management and garbage collection fundamentals

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*