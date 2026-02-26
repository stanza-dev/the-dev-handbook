---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-visitor"
---

# The Visitor Pattern

## Introduction

The Visitor pattern separates algorithms from the objects they operate on. It lets you add new operations without modifying the classes.

## Key Concepts

**Visitor**: An object that defines operations to perform on elements of a structure.

**Element**: An object that accepts a visitor via an `accept(visitor)` method.

**Double Dispatch**: The element calls the visitor method for its specific type, enabling type-specific behavior without `instanceof` checks.

## Real World Context

Compilers use Visitor to walk AST nodes (type checking, optimization, code generation). Linters, formatters (Prettier), and bundlers (Webpack plugins) all visit code structures. Serialization visitors convert object graphs to JSON/XML.

## Deep Dive

### AST Visitor

Each node type implements `accept(visitor)`, which calls the visitor method for its specific type. This double dispatch lets different visitors perform different operations on the same tree:

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

The `Calculator` visitor evaluates the expression, while the `Printer` visitor formats it as a string. Adding a new operation (e.g., optimization) requires only a new visitor object, not changes to the node classes.

### Shape Operations

Visitors work well for polymorphic calculations. Each visitor defines a method per shape type, keeping shape classes free of calculation logic:

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

Adding a new operation (e.g., `DrawVisitor`) means writing one new object. Adding a new shape (e.g., `Triangle`) means updating every existing visitor, which is the key trade-off of this pattern.

## Common Pitfalls

1. **Adding new element types is expensive** — Every visitor must add a new method for each new element type. This pattern works best when element types are stable.
2. **Circular references** — Visiting object graphs with cycles causes infinite recursion without a visited-set guard.
3. **Breaking encapsulation** — Visitors often need access to element internals, which can violate encapsulation.

## Best Practices

1. **Use Visitor when operations change frequently** — If you add new operations often but element types are stable, Visitor is ideal.
2. **Provide a default visitor** — A base visitor with no-op methods lets concrete visitors override only the methods they care about.
3. **Combine with Composite for tree traversal** — Visitor handles the operation; Composite handles the structure.

## Summary

Visitor separates operations from object structure. Add new operations without modifying classes. Use for AST processing, serialization, calculations on object hierarchies.

## Code Examples

**Visitor pattern on an AST — Calculator and Printer visitors perform different operations on the same node structure**

```javascript
class NumberNode {
  constructor(value) { this.value = value; }
  accept(visitor) { return visitor.visitNumber(this); }
}

class AddNode {
  constructor(left, right) { this.left = left; this.right = right; }
  accept(visitor) { return visitor.visitAdd(this); }
}

const Calculator = {
  visitNumber(node) { return node.value; },
  visitAdd(node) { return node.left.accept(this) + node.right.accept(this); }
};

const Printer = {
  visitNumber(node) { return String(node.value); },
  visitAdd(node) { return `(${node.left.accept(this)} + ${node.right.accept(this)})`; }
};

const expr = new AddNode(new NumberNode(1), new NumberNode(2));
expr.accept(Calculator); // 3
expr.accept(Printer);    // '(1 + 2)'
```


## Resources

- [Refactoring Guru: Visitor](https://refactoring.guru/design-patterns/visitor) — Visitor pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*