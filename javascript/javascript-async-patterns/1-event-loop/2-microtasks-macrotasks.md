---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-microtasks-macrotasks"
---

# Microtasks vs Macrotasks

## Introduction

Not all async tasks are equal. Promises use a different queue than setTimeout. Understanding the priority difference between microtasks and macrotasks explains why code execution order can be surprising—and helps you write predictable async code.

## Key Concepts

**Macrotask (Task)**: setTimeout, setInterval, setImmediate, I/O, UI rendering.

**Microtask**: Promise callbacks, queueMicrotask(), MutationObserver.

**Priority**: ALL microtasks run before the next macrotask.

## Real World Context

React's setState batching, Vue's nextTick, and many library internals use microtasks for predictable timing. Understanding the queue system helps debug async race conditions.

## Deep Dive

### The Two Queues

```javascript
console.log('1: sync');              // Sync - runs first

setTimeout(() => {                    
  console.log('4: macrotask');       // Macrotask - runs last
}, 0);

Promise.resolve().then(() => {
  console.log('3: microtask');       // Microtask - runs second
});

console.log('2: sync');              // Sync - runs first

// Output: 1: sync, 2: sync, 3: microtask, 4: macrotask
```

### Event Loop Algorithm

```
1. Execute all synchronous code (until stack is empty)
2. Execute ALL microtasks (until microtask queue is empty)
3. Execute ONE macrotask
4. Execute ALL microtasks (again!)
5. Render (if needed)
6. Repeat from step 3
```

### Microtasks Can Spawn Microtasks

```javascript
Promise.resolve().then(() => {
  console.log('microtask 1');
  Promise.resolve().then(() => {
    console.log('microtask 2');
  });
});

setTimeout(() => console.log('macrotask'), 0);

// Output: microtask 1, microtask 2, macrotask
// New microtasks run before ANY macrotask!
```

### The Danger: Infinite Microtasks

```javascript
// DON'T DO THIS - blocks forever!
function blockForever() {
  Promise.resolve().then(blockForever);
}
// blockForever(); // Page freezes!

// This is fine - macrotasks allow other work
function safer() {
  setTimeout(safer, 0);  // Other tasks can run between
}
```

### queueMicrotask()

```javascript
// Explicitly add to microtask queue
queueMicrotask(() => {
  console.log('explicit microtask');
});

// Equivalent to:
Promise.resolve().then(() => {
  console.log('promise microtask');
});
```

### Complex Example

```javascript
console.log('script start');

setTimeout(() => console.log('setTimeout'), 0);

Promise.resolve()
  .then(() => console.log('promise1'))
  .then(() => console.log('promise2'));

queueMicrotask(() => console.log('queueMicrotask'));

console.log('script end');

// Output:
// script start
// script end
// promise1
// queueMicrotask
// promise2
// setTimeout
```

## Common Pitfalls

1. **Expecting setTimeout to run before promises**: Microtasks always first.
2. **Infinite microtask loops**: Block the entire event loop.
3. **Assuming order of multiple setTimeouts**: Browser may reorder.

## Best Practices

- **Use microtasks for consistency**: Same-tick guarantees.
- **Use macrotasks to yield**: Let other events process.
- **Avoid recursive microtasks**: Can freeze the page.
- **Test with DevTools**: Profile to see queue timing.

## Summary

Microtasks (Promises, queueMicrotask) run before macrotasks (setTimeout). ALL microtasks drain before the next macrotask. Microtasks can schedule more microtasks. Be careful of infinite microtask loops—they block forever.

## Resources

- [MDN: queueMicrotask()](https://developer.mozilla.org/en-US/docs/Web/API/queueMicrotask) — queueMicrotask API reference
- [MDN: Using microtasks](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API/Microtask_guide) — Guide to using microtasks

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*