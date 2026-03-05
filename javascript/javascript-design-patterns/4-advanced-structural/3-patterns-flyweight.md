---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-flyweight"
---

# The Flyweight Pattern

## Introduction

The Flyweight pattern minimizes memory by sharing common data between similar objects. It separates intrinsic (shared) state from extrinsic (unique) state.

## Key Concepts

**Intrinsic State**: Shared, immutable data stored in the flyweight (e.g., font, color).

**Extrinsic State**: Unique data kept outside the flyweight (e.g., position, character).

**Flyweight Factory**: Creates and manages shared flyweight instances.

**Object Pool**: The cache of reusable flyweight objects.

## Real World Context

Text editors share font/style objects across millions of characters. Game engines share sprite textures for identical objects. String interning in JavaScript (identical strings share memory) is a language-level flyweight. Icon libraries cache loaded SVGs.

## Deep Dive

### Character Rendering

In a text editor, font/size/color (intrinsic state) is shared across characters, while each character's value and position (extrinsic state) are unique:

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

Instead of each of the thousands of characters storing its own font, size, and color, they all reference a single shared `CharacterType` instance from the factory cache.

### Icon Cache

An icon cache is a simplified flyweight that stores loaded icons by name, avoiding repeated expensive I/O for the same icon:

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

The first call to `getIcon('star')` loads the icon from disk. Every subsequent call returns the cached reference, making repeated renders essentially free.

## Common Pitfalls

1. **Premature optimization** — Don't use Flyweight until profiling shows memory is actually the problem. The added complexity has a cost.
2. **Mutable intrinsic state** — If shared state is mutated, all flyweight users are affected. Keep intrinsic state immutable.
3. **Key collisions in factory** — Make sure the cache key uniquely identifies the flyweight or you'll share the wrong object.

## Best Practices

1. **Separate intrinsic from extrinsic clearly** — Document which state is shared and which is per-instance.
2. **Use a factory with `Map` for caching** — A `Map` keyed by the intrinsic properties ensures each unique combination is created only once.
3. **Measure before and after** — Flyweight is justified when you can show measurable memory reduction.

## Summary

Flyweight shares common state to reduce memory. Separate intrinsic (shared) from extrinsic (unique) state. Use for text rendering, icons, game objects.

## Code Examples

**Flyweight factory for text styles — thousands of characters share the same cached CharStyle object**

```javascript
class CharStyle {
  constructor(font, size, color) {
    this.font = font; this.size = size; this.color = color;
  }
}

class StyleFactory {
  #cache = new Map();
  get(font, size, color) {
    const key = `${font}-${size}-${color}`;
    if (!this.#cache.has(key)) {
      this.#cache.set(key, new CharStyle(font, size, color));
    }
    return this.#cache.get(key);
  }
}

const factory = new StyleFactory();
const style = factory.get('Arial', 12, 'black');
// 10,000 characters can share this single style object
console.log(factory.get('Arial', 12, 'black') === style); // true
```


## Resources

- [Refactoring Guru: Flyweight](https://refactoring.guru/design-patterns/flyweight) — Flyweight pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*