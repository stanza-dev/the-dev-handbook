---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-generators"
---

# Generator Functions

## Introduction

Generators are functions that can pause and resume execution. Unlike regular functions that run to completion, generators yield values one at a time, maintaining their state between calls. They're perfect for lazy sequences, state machines, and async flow control.

## Key Concepts

**Generator Function**: Declared with function* syntax, returns a generator object.

**yield**: Pauses execution and returns a value to the caller.

**Generator Object**: An iterator that can be paused/resumed.

## Deep Dive

### Basic Syntax

```javascript
function* generateSequence() {
  yield 1;
  yield 2;
  yield 3;
  return 'done';  // Final value, done: true
}

const gen = generateSequence();
gen.next();  // { value: 1, done: false }
gen.next();  // { value: 2, done: false }
gen.next();  // { value: 3, done: false }
gen.next();  // { value: 'done', done: true }
gen.next();  // { value: undefined, done: true }
```

### Generators are Iterable

```javascript
function* range(start, end) {
  for (let i = start; i <= end; i++) {
    yield i;
  }
}

for (const n of range(1, 5)) {
  console.log(n);  // 1, 2, 3, 4, 5
}

[...range(1, 5)];  // [1, 2, 3, 4, 5]
```

### Passing Values Into Generators

```javascript
function* calculator() {
  const a = yield 'Enter first number';
  const b = yield 'Enter second number';
  return a + b;
}

const calc = calculator();
calc.next();      // { value: 'Enter first number', done: false }
calc.next(10);    // { value: 'Enter second number', done: false }
calc.next(5);     // { value: 15, done: true }
```

### yield* Delegation

```javascript
function* letters() {
  yield 'a';
  yield 'b';
}

function* numbers() {
  yield 1;
  yield 2;
}

function* combined() {
  yield* letters();   // Delegate to letters()
  yield* numbers();   // Delegate to numbers()
  yield 'end';
}

[...combined()];  // ['a', 'b', 1, 2, 'end']
```

### Infinite Sequences

```javascript
function* fibonacci() {
  let [prev, curr] = [0, 1];
  while (true) {
    yield curr;
    [prev, curr] = [curr, prev + curr];
  }
}

const fib = fibonacci();
fib.next().value;  // 1
fib.next().value;  // 1
fib.next().value;  // 2
fib.next().value;  // 3
// Goes forever - use with care!
```

## Common Pitfalls

1. **Spreading infinite generators**: Crashes your program.
2. **Forgetting function***: Without *, it's a normal function.
3. **Generator consumed after iteration**: Can't reuse; create new one.

## Best Practices

- **Use generators for lazy sequences**: Memory efficient for large data.
- **Use for state machines**: Maintain state between yields.
- **Prefer async/await over generator-based async**: Unless you need fine control.

## Summary

Generators (function*) yield values on demand. The generator object has next() and is iterable. yield* delegates to other iterables. Generators are great for lazy sequences and maintaining state. Be careful with infinite generators.

## Code Examples

**Basic Syntax**

```javascript
function* generateSequence() {
  yield 1;
  yield 2;
  yield 3;
  return 'done';  // Final value, done: true
}

const gen = generateSequence();
gen.next();  // { value: 1, done: false }
gen.next();  // { value: 2, done: false }
gen.next();  // { value: 3, done: false }
gen.next();  // { value: 'done', done: true }
gen.next();  // { value: undefined, done: true }
```

**Generators are Iterable**

```javascript
function* range(start, end) {
  for (let i = start; i <= end; i++) {
    yield i;
  }
}

for (const n of range(1, 5)) {
  console.log(n);  // 1, 2, 3, 4, 5
}

[...range(1, 5)];  // [1, 2, 3, 4, 5]
```


## Resources

- [MDN: function*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*) — Generator function reference
- [MDN: yield](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield) — yield operator reference

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*