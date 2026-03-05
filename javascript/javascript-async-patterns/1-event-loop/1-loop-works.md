---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-how-event-loop-works"
---

# The Call Stack and Event Loop

## Introduction

JavaScript runs on a single thread, yet it handles thousands of concurrent operations—network requests, timers, user interactions. The secret is the event loop, a clever mechanism that orchestrates async execution. Understanding it is essential for writing efficient, bug-free async code.

## Key Concepts

**Call Stack**: LIFO structure tracking function execution. Only one thing runs at a time.

**Event Loop**: Continuously checks if the call stack is empty, then processes queued tasks.

**Non-blocking**: Operations that don't prevent other code from running.

## Real World Context

When a fetch request takes 500ms, your UI doesn't freeze. When a click handler runs, other events queue up. Understanding this prevents blocking bugs and helps debug timing issues in complex applications.

## Deep Dive

### The Call Stack

```javascript
function multiply(a, b) {
  return a * b;
}

function square(n) {
  return multiply(n, n);
}

function printSquare(n) {
  const result = square(n);
  console.log(result);
}

printSquare(4);
// Stack: [printSquare] -> [square, printSquare] 
//     -> [multiply, square, printSquare] -> pops down
```

### How Async Works

```javascript
console.log('Start');           // 1. Runs immediately

setTimeout(() => {
  console.log('Timeout');       // 4. Runs after stack is empty
}, 0);

console.log('End');             // 2. Runs immediately

// Output: Start, End, Timeout
// Even 0ms timeout goes to queue!
```

### The Mechanism

1. **Call Stack**: Executes synchronous code
2. **Web APIs**: Handle async operations (timers, fetch, events)
3. **Task Queue**: Stores callbacks ready to run
4. **Event Loop**: Moves tasks from queue to stack when empty

```javascript
// Visualization:
// 
// [Call Stack]      [Web APIs]       [Task Queue]
//    main()    -->  setTimeout() -->  callback()
//              -->  fetch()       -->  .then()
//              -->  onClick      -->  handler()
//
// Event Loop: Stack empty? Move from Queue to Stack
```

### Why 0ms Timeout Isn't Immediate

```javascript
setTimeout(() => console.log('timeout'), 0);
console.log('sync');

// Output: sync, timeout
// setTimeout ALWAYS goes through the queue
// minimum delay is ~4ms in most browsers
```

## Common Pitfalls

1. **Assuming setTimeout(0) is immediate**: It still goes through the queue.
2. **Blocking with synchronous loops**: Freezes the entire page.
3. **Expecting consistent timing**: Busy stack delays queue processing.

## Best Practices

- **Keep synchronous work short**: Long tasks block everything.
- **Use async/await for clarity**: Easier to reason about than callbacks.
- **Profile with DevTools**: Performance tab shows long tasks.
- **Break up heavy computation**: Use `requestIdleCallback` or workers.

## Summary

JavaScript is single-threaded with one call stack. Async operations use Web APIs and queues. The event loop moves queued callbacks to the stack when empty. Even 0ms timeouts go through the queue. Long synchronous code blocks everything.

## Code Examples

**The Call Stack**

```javascript
function multiply(a, b) {
  return a * b;
}

function square(n) {
  return multiply(n, n);
}

function printSquare(n) {
  const result = square(n);
  console.log(result);
}

printSquare(4);
// Stack: [printSquare] -> [square, printSquare] 
//     -> [multiply, square, printSquare] -> pops down
```

**How Async Works**

```javascript
console.log('Start');           // 1. Runs immediately

setTimeout(() => {
  console.log('Timeout');       // 4. Runs after stack is empty
}, 0);

console.log('End');             // 2. Runs immediately

// Output: Start, End, Timeout
// Even 0ms timeout goes to queue!
```


## Resources

- [MDN: Event Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Event_loop) — Concurrency model and event loop
- [JSConf: What the heck is the event loop anyway?](https://www.youtube.com/watch?v=8aGhZQkoFbQ) — Philip Roberts' famous event loop visualization talk

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*