---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-singleton"
---

# The Singleton Pattern

## Introduction

The Singleton pattern ensures a class has only one instance and provides global access to it. While controversial in some circles, Singletons are useful for shared resources like configurations, connection pools, and logging services.

## Key Concepts

**Singleton**: A class that guarantees only one instance exists.

**Lazy Initialization**: Creating the instance only when first needed.

**Global Access Point**: Single entry point to the instance.

## Real World Context

Database connections, application configuration, logging services, caches—Singletons manage shared resources efficiently. Many frameworks use Singletons internally.

## Deep Dive

### Classic Implementation (IIFE)

```javascript
const Singleton = (function() {
  let instance;
  
  function createInstance() {
    return {
      config: {},
      setConfig(key, value) {
        this.config[key] = value;
      },
      getConfig(key) {
        return this.config[key];
      }
    };
  }
  
  return {
    getInstance() {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();

const a = Singleton.getInstance();
const b = Singleton.getInstance();
a === b;  // true - same instance!
```

### ES6 Class Implementation

```javascript
class Database {
  static #instance = null;
  
  constructor() {
    if (Database.#instance) {
      return Database.#instance;
    }
    this.connection = null;
    Database.#instance = this;
  }
  
  static getInstance() {
    if (!Database.#instance) {
      Database.#instance = new Database();
    }
    return Database.#instance;
  }
  
  connect(config) {
    if (!this.connection) {
      this.connection = createConnection(config);
    }
    return this.connection;
  }
}

const db1 = Database.getInstance();
const db2 = Database.getInstance();
db1 === db2;  // true
```

### Module Pattern (Simplest)

```javascript
// config.js
const config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000
};

export default config;

// ES modules are singletons by nature!
// Every import gets the same object
```

## Common Pitfalls

1. **Global state issues**: Singletons can make testing difficult.
2. **Hidden dependencies**: Code depends on global state.
3. **Thread safety (Node.js)**: Ensure atomic initialization.

## Best Practices

- **Use ES modules for simple cases**: Natural singletons.
- **Consider dependency injection**: Instead of global access.
- **Make singletons configurable**: Allow resetting for tests.
- **Document singleton dependencies**: Make usage explicit.

## Summary

Singletons ensure one instance exists globally. ES modules are natural singletons. Use for shared resources like config, logging, and connection pools. Be aware of testing challenges with global state.

## Resources

- [Patterns.dev: Singleton Pattern](https://www.patterns.dev/posts/singleton-pattern/) — In-depth Singleton pattern explanation

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*