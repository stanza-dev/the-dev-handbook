---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-composite"
---

# The Composite Pattern

## Introduction

The Composite pattern composes objects into tree structures to represent part-whole hierarchies. It lets clients treat individual objects and compositions uniformly.

## Deep Dive

### File System

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

### UI Component Tree

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

## Summary

Composite treats individual objects and groups uniformly. Use for trees like file systems, UI components, organization charts.

## Resources

- [Refactoring Guru: Composite](https://refactoring.guru/design-patterns/composite) — Composite pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*