---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-facade-pattern"
---

# The Facade Pattern

## Introduction

The Facade pattern provides a simplified interface to a complex subsystem. It hides internal complexity and exposes a clean API.

## Key Concepts

**Facade**: A simplified interface to a complex subsystem.

**Subsystem**: The collection of classes/modules the facade wraps.

**Convenience Methods**: High-level methods that orchestrate multiple subsystem calls.

## Real World Context

jQuery is a classic Facade over the DOM API. Axios facades over `fetch` with interceptors and defaults. ORMs like Prisma facade complex SQL into simple method calls. Any SDK that wraps a REST API is a facade.

## Deep Dive

### HTTP Client Facade

This API client hides `fetch` configuration, JSON parsing, and header management behind simple `get()` and `post()` methods:

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

Consumers call `api.get('/users')` instead of manually configuring `fetch` with headers, base URLs, and JSON parsing every time.

### Configuration Facade

A configuration facade consolidates multiple data sources (files, environment variables, defaults) behind a single `get()` method:

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

The `load()` method handles the complexity of merging multiple sources, while `get()` provides a clean key-value lookup with a fallback. Consumers never interact with the raw data sources.

## Common Pitfalls

1. **God facade** — Putting every subsystem method on the facade defeats its purpose. Only expose common operations.
2. **Hiding too much** — If advanced users can never reach the subsystem directly, the facade becomes a bottleneck.
3. **Tight coupling** — The facade should depend on abstractions, not concrete implementations of the subsystem.

## Best Practices

1. **Expose the subsystem for advanced use** — Provide escape hatches so power users can bypass the facade when needed.
2. **Keep facades stateless** — Facades should delegate to the subsystem, not maintain their own state.
3. **Version facades alongside APIs** — When the underlying API changes, update the facade to maintain backward compatibility.

## Summary

Facades simplify complex subsystems with clean APIs. Use for HTTP clients, configuration, media players. Keep them focused on common use cases.

## Code Examples

**HTTP client Facade — hides fetch configuration, auth headers, and error handling behind clean get/post methods**

```javascript
class ApiClient {
  #baseUrl;
  #headers;

  constructor(baseUrl, token) {
    this.#baseUrl = baseUrl;
    this.#headers = {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`
    };
  }

  async get(path) {
    const res = await fetch(`${this.#baseUrl}${path}`, { headers: this.#headers });
    if (!res.ok) throw new Error(`GET ${path}: ${res.status}`);
    return res.json();
  }

  async post(path, data) {
    const res = await fetch(`${this.#baseUrl}${path}`, {
      method: 'POST', headers: this.#headers, body: JSON.stringify(data)
    });
    if (!res.ok) throw new Error(`POST ${path}: ${res.status}`);
    return res.json();
  }
}
```


## Resources

- [Refactoring Guru: Facade](https://refactoring.guru/design-patterns/facade) — Facade pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*