---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-memory-leaks"
---

# Common Memory Leaks

## Introduction

Memory leaks occur when objects are no longer needed but can't be garbage collected because something still references them.

## Deep Dive

### Accidental Globals

```javascript
function leak() {
  leaked = 'oops';  // No var/let/const → global!
}

// Fix: Use strict mode
'use strict';
function noLeak() {
  leaked = 'oops';  // ReferenceError!
}
```

### Forgotten Timers

```javascript
// Leak: timer keeps reference
const data = heavyObject;
setInterval(() => {
  process(data);  // data can't be collected
}, 1000);

// Fix: Clear when done
const id = setInterval(callback, 1000);
clearInterval(id);
```

### Detached DOM Nodes

```javascript
// Leak: reference to removed element
const button = document.getElementById('btn');
document.body.removeChild(button);
// button variable still holds reference!

// Fix: Nullify references
let button = document.getElementById('btn');
button.remove();
button = null;
```

### Closures

```javascript
function createHandler() {
  const bigData = new Array(1000000);
  return function handler() {
    // bigData captured in closure, never freed
  };
}

// Fix: Only capture what you need
function createHandler() {
  const bigData = new Array(1000000);
  const needed = bigData.length;
  return function handler() {
    console.log(needed);  // Only length captured
  };
}
```

## Summary

Common leaks: accidental globals, forgotten timers, detached DOM nodes, closures. Use strict mode, clear timers, nullify DOM references, minimize closure captures.

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*