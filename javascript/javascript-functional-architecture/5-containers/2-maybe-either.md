---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-maybe-either"
---

# Maybe & Either

## Introduction

Maybe and Either are functors that handle absence and errors safely. They eliminate null checks and try/catch boilerplate.

## Deep Dive

### Maybe (Option)

```javascript
const Just = x => ({
  map: f => Just(f(x)),
  fold: (onNothing, onJust) => onJust(x),
  isNothing: false
});

const Nothing = () => ({
  map: f => Nothing(),
  fold: (onNothing, onJust) => onNothing(),
  isNothing: true
});

const Maybe = x => x == null ? Nothing() : Just(x);

// Safe property access
Maybe(user)
  .map(u => u.address)
  .map(a => a.street)
  .fold(
    () => 'No street',
    street => street
  );
```

### Either (Result)

```javascript
const Right = x => ({
  map: f => Right(f(x)),
  fold: (onLeft, onRight) => onRight(x),
  isLeft: false
});

const Left = x => ({
  map: f => Left(x),  // Doesn't transform!
  fold: (onLeft, onRight) => onLeft(x),
  isLeft: true
});

// Error handling
const parseJSON = str => {
  try {
    return Right(JSON.parse(str));
  } catch (e) {
    return Left(e.message);
  }
};

parseJSON('{"a":1}')
  .map(obj => obj.a)
  .fold(
    err => console.error(err),
    val => console.log(val)
  );
```

## Summary

Maybe handles null/undefined safely. Either handles success/error. Left short-circuits (doesn't map). fold extracts with handling for both cases.

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*