---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-practical-generators"
---

# Practical Generator Patterns

## Introduction

Generators shine in specific scenarios: lazy evaluation, infinite sequences, state machines, and composable data transformations. Understanding these patterns helps you recognize when generators are the right tool.

## Deep Dive

### Lazy Evaluation

```javascript
// Only compute values when needed
function* lazyMap(iterable, fn) {
  for (const item of iterable) {
    yield fn(item);
  }
}

function* lazyFilter(iterable, predicate) {
  for (const item of iterable) {
    if (predicate(item)) yield item;
  }
}

function* lazyTake(iterable, n) {
  let count = 0;
  for (const item of iterable) {
    if (count++ >= n) return;
    yield item;
  }
}

// Compose without creating intermediate arrays!
function* hugeData() {
  for (let i = 0; i < 1000000; i++) yield i;
}

const result = lazyTake(
  lazyFilter(
    lazyMap(hugeData(), x => x * 2),
    x => x % 3 === 0
  ),
  5
);

[...result];  // First 5 even numbers divisible by 3
```

### State Machine

```javascript
function* trafficLight() {
  while (true) {
    yield 'green';
    yield 'yellow';
    yield 'red';
  }
}

const light = trafficLight();
light.next().value;  // 'green'
light.next().value;  // 'yellow'
light.next().value;  // 'red'
light.next().value;  // 'green' (cycles)
```

### ID Generator

```javascript
function* idGenerator(prefix = '') {
  let id = 0;
  while (true) {
    yield `${prefix}${++id}`;
  }
}

const userIds = idGenerator('user_');
userIds.next().value;  // 'user_1'
userIds.next().value;  // 'user_2'
```

### Cooperative Multitasking

```javascript
function* longTask() {
  for (let i = 0; i < 1000; i++) {
    doExpensiveWork(i);
    if (i % 10 === 0) yield;  // Yield control periodically
  }
}

// Run without blocking
function runCooperatively(gen) {
  const iterator = gen();
  function step() {
    const { done } = iterator.next();
    if (!done) {
      setTimeout(step, 0);  // Let other tasks run
    }
  }
  step();
}
```

## Summary

Generators enable lazy evaluation, state machines, ID generation, and cooperative multitasking. Use them when you need on-demand computation, infinite sequences, or fine-grained control over execution flow.

## Resources

- [MDN: Iterators and generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_generators) — Complete guide to iterators and generators

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*