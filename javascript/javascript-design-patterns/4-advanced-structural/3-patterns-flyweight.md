---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-flyweight"
---

# The Flyweight Pattern

## Introduction

The Flyweight pattern minimizes memory by sharing common data between similar objects. It separates intrinsic (shared) state from extrinsic (unique) state.

## Deep Dive

### Character Rendering

```javascript
// Flyweight: shared character data
class CharacterType {
  constructor(font, size, color) {
    this.font = font;
    this.size = size;
    this.color = color;
  }
}

class CharacterTypeFactory {
  #types = new Map();
  
  getType(font, size, color) {
    const key = `${font}-${size}-${color}`;
    if (!this.#types.has(key)) {
      this.#types.set(key, new CharacterType(font, size, color));
    }
    return this.#types.get(key);
  }
}

// Context: unique character data
class Character {
  constructor(char, type, x, y) {
    this.char = char;  // Extrinsic
    this.type = type;  // Intrinsic (shared)
    this.x = x;
    this.y = y;
  }
}

const factory = new CharacterTypeFactory();
const type = factory.getType('Arial', 12, 'black');
// All characters share the same type object
```

### Icon Cache

```javascript
class IconCache {
  #cache = new Map();
  
  getIcon(name) {
    if (!this.#cache.has(name)) {
      this.#cache.set(name, loadIcon(name)); // Expensive
    }
    return this.#cache.get(name);
  }
}
```

## Summary

Flyweight shares common state to reduce memory. Separate intrinsic (shared) from extrinsic (unique) state. Use for text rendering, icons, game objects.

## Resources

- [Refactoring Guru: Flyweight](https://refactoring.guru/design-patterns/flyweight) — Flyweight pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*