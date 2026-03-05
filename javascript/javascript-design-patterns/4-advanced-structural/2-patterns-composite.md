---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-composite"
---

# The Composite Pattern

## Introduction

The Composite pattern composes objects into tree structures to represent part-whole hierarchies. It lets clients treat individual objects and compositions uniformly.

## Key Concepts

**Component**: The common interface shared by leaves and composites.

**Leaf**: An individual object with no children.

**Composite**: A container that holds children and implements the same interface as leaves.

**Uniform Treatment**: Clients use the same methods on both individual objects and groups.

## Real World Context

The DOM is a composite tree—`Element` and `Text` nodes share common interfaces. React component trees, file systems (files and folders), and organizational charts all follow this pattern. JSON data structures are natural composites.

## Deep Dive

### File System

Both `File` and `Directory` implement the same `getSize()` and `print()` interface. A directory recursively aggregates its children's sizes:

```javascript
class File {
  constructor(name, size) {
    this.name = name;
    this.size = size;
  }
  
  getSize() { return this.size; }
  print(indent = '') { console.log(`${indent}${this.name}`); }
}

class Directory {
  constructor(name) {
    this.name = name;
    this.children = [];
  }
  
  add(child) { this.children.push(child); }
  
  getSize() {
    return this.children.reduce((sum, c) => sum + c.getSize(), 0);
  }
  
  print(indent = '') {
    console.log(`${indent}${this.name}/`);
    this.children.forEach(c => c.print(indent + '  '));
  }
}

const root = new Directory('root');
root.add(new File('file1.txt', 100));
const sub = new Directory('subdir');
sub.add(new File('file2.txt', 200));
root.add(sub);
root.getSize(); // 300
```

Calling `root.getSize()` triggers a recursive traversal. Files return their own size, and directories sum their children, producing the total for the entire subtree.

### UI Component Tree

The Composite pattern maps naturally to UI component trees. A `Container` renders its children, while a `Button` renders itself as a leaf:

```javascript
class Component {
  render() { throw new Error('Override me'); }
}

class Button extends Component {
  render() { return '<button>Click</button>'; }
}

class Container extends Component {
  children = [];
  add(child) { this.children.push(child); }
  render() {
    return `<div>${this.children.map(c => c.render()).join('')}</div>`;
  }
}
```

Both `Button` and `Container` implement `render()`. Client code can call `render()` on any component without knowing whether it is a leaf or a composite.

## Common Pitfalls

1. **Type-checking children** — Avoid `instanceof` checks on composite members; the pattern's value is uniform treatment.
2. **Missing base case in recursive operations** — Operations on composites are recursive; forgetting the leaf case causes infinite loops.
3. **Overly deep nesting** — Deep composite trees can cause stack overflows during recursive traversal.

## Best Practices

1. **Keep the component interface minimal** — Only include operations that make sense for both leaves and composites.
2. **Use iterators for traversal** — Decouple traversal logic from the composite structure.
3. **Consider the Visitor pattern for operations** — When you need many different operations on the tree, Visitor avoids polluting the component interface.

## Summary

Composite treats individual objects and groups uniformly. Use for trees like file systems, UI components, organization charts.

## Code Examples

**Composite file system — File and Directory both implement getSize(), enabling uniform treatment of leaves and branches**

```javascript
class File {
  constructor(name, size) { this.name = name; this.size = size; }
  getSize() { return this.size; }
}

class Directory {
  constructor(name) { this.name = name; this.children = []; }
  add(child) { this.children.push(child); return this; }
  getSize() {
    return this.children.reduce((sum, c) => sum + c.getSize(), 0);
  }
}

const root = new Directory('root')
  .add(new File('readme.md', 100))
  .add(new Directory('src')
    .add(new File('index.js', 500))
    .add(new File('utils.js', 300)));

root.getSize(); // 900 — works uniformly on files and directories
```


## Resources

- [Refactoring Guru: Composite](https://refactoring.guru/design-patterns/composite) — Composite pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*