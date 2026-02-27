---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-facade-pattern"
---

# The Facade Pattern

## Introduction

The Facade pattern provides a simplified interface to a complex subsystem. It hides internal complexity and exposes a clean API.

## Deep Dive

### HTTP Client Facade

```javascript
class ApiClient {
  #baseUrl;
  
  constructor(baseUrl) {
    this.#baseUrl = baseUrl;
  }
  
  async get(path) {
    const res = await fetch(`${this.#baseUrl}${path}`);
    return res.json();
  }
  
  async post(path, data) {
    const res = await fetch(`${this.#baseUrl}${path}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    return res.json();
  }
}

const api = new ApiClient('https://api.example.com');
const users = await api.get('/users');
```

### Configuration Facade

```javascript
class Config {
  static #data = {};
  
  static async load() {
    this.#data = await loadFromMultipleSources();
  }
  
  static get(key, fallback) {
    return this.#data[key] ?? fallback;
  }
}
```

## Summary

Facades simplify complex subsystems with clean APIs. Use for HTTP clients, configuration, media players. Keep them focused on common use cases.

## Resources

- [Refactoring Guru: Facade](https://refactoring.guru/design-patterns/facade) — Facade pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*