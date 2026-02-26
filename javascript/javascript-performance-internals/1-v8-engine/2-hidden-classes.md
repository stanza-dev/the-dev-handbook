---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-hidden-classes"
---

## Introduction

JavaScript objects are dynamic: you can add, delete, and modify properties at any time. Yet V8 accesses object properties nearly as fast as a C++ struct. The secret is **hidden classes** (also called Shapes or Maps internally), a mechanism that gives structure to dynamic objects so the engine can optimize property access.

Every time you create an object or add a property, V8 assigns or transitions to a hidden class that describes the object's layout. Combined with **inline caching**, this allows V8 to skip expensive dictionary lookups and access properties at fixed memory offsets.

## Key Concepts

- **Hidden Classes/Shapes**: Internal structures V8 creates to describe the layout of JavaScript objects. They map property names to fixed memory offsets, enabling fast access. In V8 source code, these are called "Maps" (not to be confused with the JavaScript `Map` object).

- **Inline Caching**: An optimization where V8 remembers the hidden class seen at a property access site and caches the memory offset. On subsequent accesses, if the hidden class matches, V8 reads the value directly without a lookup.

- **Transition Chains**: When you add a property to an object, V8 creates a transition from the current hidden class to a new one. These transitions form a chain that V8 reuses for objects initialized the same way.

- **Monomorphic/Polymorphic/Megamorphic**: Inline cache states. A monomorphic site always sees one hidden class (fastest). A polymorphic site sees 2-4 classes (still fast, uses a small lookup table). A megamorphic site sees 5+ classes (falls back to slow hash table lookup).

## Real World Context

Hidden classes explain why constructor functions and classes are faster than ad-hoc object literals in hot paths. When every instance of a class initializes properties in the same order, all instances share the same hidden class chain, making inline caches monomorphic.

This is also why popular style guides recommend initializing all properties in constructors: frameworks like Vue.js and React rely on consistent object shapes for efficient reactivity tracking and virtual DOM diffing.

## Deep Dive

Let us trace how V8 creates hidden classes step by step:

```javascript
function Point(x, y) {
  this.x = x;  // Step 1: Hidden class C0 (empty) → C1 (has x at offset 0)
  this.y = y;  // Step 2: Hidden class C1 → C2 (has x at 0, y at offset 1)
}

const p1 = new Point(1, 2);  // Uses hidden class C2
const p2 = new Point(3, 4);  // Reuses hidden class C2 (same shape!)
```

Both `p1` and `p2` share hidden class `C2`. When V8 encounters `p1.x`, it caches the fact that objects with hidden class `C2` store `x` at offset 0. Next time it sees `p2.x`, it recognizes the same hidden class and reads offset 0 directly, no lookup needed.

Now watch what happens when you break the pattern:

```javascript
const p3 = new Point(5, 6);
p3.z = 7;  // Transition to NEW hidden class C3 (has x, y, z)

const p4 = {};
p4.y = 1;  // Hidden class: {} → {y}
p4.x = 2;  // Hidden class: {y} → {y, x}
// p4 has DIFFERENT hidden class than p1, even though same properties!
```

Property addition order matters. `p4` has properties `x` and `y` just like `p1`, but because they were added in different order (`y` first, then `x`), they have completely different hidden classes.

Inline caching at a property access site:

```javascript
function getX(point) {
  return point.x;  // IC site
}

getX(p1); // IC: monomorphic (C2 → offset 0)
getX(p2); // IC: still monomorphic (same C2)
getX(p4); // IC: polymorphic (C2 or C_p4)
getX({ x: 1, y: 2, z: 3 }); // IC: potentially megamorphic
```

## Common Pitfalls

- **Adding properties in different orders**: Creating objects with the same properties but in different initialization order results in different hidden classes, defeating inline caching.

- **Deleting properties**: The `delete` operator forces V8 to abandon the hidden class system for that object and fall back to a slow dictionary mode. Prefer setting properties to `undefined` if you need to "remove" a value.

- **Dynamic property names**: Using `obj[dynamicKey] = value` with many different keys creates many transitions and can make objects go to dictionary mode.

## Best Practices

- **Initialize all properties in constructors**: Declare every property in the constructor or class, even if the initial value is `undefined`. This ensures all instances share the same hidden class.

- **Use consistent object shapes**: When creating objects in a loop or factory function, always add properties in the same order with the same types.

- **Avoid `delete`**: Instead of deleting properties, set them to `null` or `undefined` to preserve the hidden class.

## Summary

Hidden classes give V8 a way to treat dynamic JavaScript objects like static structs under the hood. By tracking property layouts and caching memory offsets via inline caches, V8 achieves near-native property access speed. Consistent object initialization order ensures objects share hidden classes, keeping inline caches monomorphic and property access fast.

## Code Examples

**How property initialization order affects hidden classes**

```javascript
// Same hidden class - fast property access
function Point(x, y) {
  this.x = x; // Transition: {} → {x}
  this.y = y; // Transition: {x} → {x, y}
}
const p1 = new Point(1, 2);
const p2 = new Point(3, 4); // Same hidden class as p1

// Different hidden class - slower
const p3 = new Point(5, 6);
p3.z = 7; // New transition → {x, y, z}
```


## Resources

- [V8 Blog: Fast Properties](https://v8.dev/blog/fast-properties) — How V8 handles JavaScript object properties

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*