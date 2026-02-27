---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-hidden-classes"
---

# Hidden Classes & Shapes

## Introduction

JavaScript is dynamically typed, but engines use hidden classes (called Shapes in SpiderMonkey, Maps in V8) to optimize property access. Understanding them is key to writing fast code.

## Deep Dive

### How Hidden Classes Work

```javascript
function Point(x, y) {
  this.x = x;  // Creates hidden class C1
  this.y = y;  // Transitions to C2
}

const p1 = new Point(1, 2);
const p2 = new Point(3, 4);
// p1 and p2 share the same hidden class C2
// Property access is optimized!
```

### Breaking Hidden Classes

```javascript
// Bad - different hidden classes
function createPoint() {
  const p = {};
  p.x = 1;  // C1
  p.y = 2;  // C2
  return p;
}

function createPoint2() {
  const p = {};
  p.y = 2;  // Different C1!
  p.x = 1;  // Different C2!
  return p;
}

// p1 and p2 have different hidden classes
const p1 = createPoint();
const p2 = createPoint2();
```

### Inline Caching

```javascript
function getX(point) {
  return point.x;  // IC site
}

// Monomorphic (fast): Same shape every time
getX({ x: 1, y: 2 });
getX({ x: 3, y: 4 });

// Polymorphic (slower): Few shapes
getX({ x: 1, y: 2 });
getX({ x: 1, z: 3 });  // Different shape!

// Megamorphic (slowest): Many shapes
// Falls back to dictionary lookup
```

### Property Order Matters

```javascript
// Same hidden class
const a = { x: 1, y: 2 };
const b = { x: 3, y: 4 };

// Different hidden classes!
const c = { y: 1, x: 2 };  // Different order
const d = { x: 1, y: 2, z: 3 };  // Extra property
```

## Best Practices

- **Initialize properties in consistent order**: Constructor or object literal.
- **Don't add properties later**: Define all in constructor.
- **Don't delete properties**: Changes shape.
- **Use classes or factory functions**: Ensure consistent shapes.

## Summary

Hidden classes optimize property access. Same shape = fast inline caching. Add properties in consistent order. Don't delete or add properties after creation.

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*