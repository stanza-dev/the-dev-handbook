---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-visitor"
---

# The Visitor Pattern

## Introduction

The Visitor pattern separates algorithms from the objects they operate on. It lets you add new operations without modifying the classes.

## Deep Dive

### AST Visitor

```javascript
class NumberNode {
  constructor(value) { this.value = value; }
  accept(visitor) { return visitor.visitNumber(this); }
}

class AddNode {
  constructor(left, right) {
    this.left = left;
    this.right = right;
  }
  accept(visitor) { return visitor.visitAdd(this); }
}

// Visitor: calculate result
const Calculator = {
  visitNumber(node) { return node.value; },
  visitAdd(node) {
    return node.left.accept(this) + node.right.accept(this);
  }
};

// Visitor: print expression
const Printer = {
  visitNumber(node) { return String(node.value); },
  visitAdd(node) {
    return `(${node.left.accept(this)} + ${node.right.accept(this)})`;
  }
};

// 1 + 2
const expr = new AddNode(new NumberNode(1), new NumberNode(2));
expr.accept(Calculator); // 3
expr.accept(Printer);    // '(1 + 2)'
```

### Shape Operations

```javascript
const AreaVisitor = {
  visitCircle(c) { return Math.PI * c.radius ** 2; },
  visitRectangle(r) { return r.width * r.height; }
};

const PerimeterVisitor = {
  visitCircle(c) { return 2 * Math.PI * c.radius; },
  visitRectangle(r) { return 2 * (r.width + r.height); }
};
```

## Summary

Visitor separates operations from object structure. Add new operations without modifying classes. Use for AST processing, serialization, calculations on object hierarchies.

## Resources

- [Refactoring Guru: Visitor](https://refactoring.guru/design-patterns/visitor) — Visitor pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*