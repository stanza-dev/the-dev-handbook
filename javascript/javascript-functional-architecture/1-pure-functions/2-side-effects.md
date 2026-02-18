---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-side-effects"
---

# Managing Side Effects

## Introduction

Real programs need side effects—I/O, DOM updates, network calls. The key is isolating them from pure logic.

## Deep Dive

### Identify Side Effects

```javascript
// Side effects:
// - Mutating arguments
// - Mutating external state
// - Console logging
// - DOM manipulation
// - Network requests
// - Reading/writing files
// - Getting current date/time
// - Random number generation
```

### Push Effects to the Edges

```javascript
// Bad: Mixed logic and side effects
function processUser(id) {
  const user = fetch(`/users/${id}`);  // Side effect
  const name = user.name.toUpperCase();  // Logic
  document.body.innerHTML = name;  // Side effect
}

// Good: Separate concerns
const formatName = name => name.toUpperCase();  // Pure

async function processUser(id) {
  const user = await fetchUser(id);  // Effect at edge
  const name = formatName(user.name);  // Pure logic
  render(name);  // Effect at edge
}
```

### Dependency Injection

```javascript
// Hard to test - hidden dependency
function getUser(id) {
  return fetch(`/users/${id}`);  // Hardcoded side effect
}

// Testable - inject dependency
function getUser(id, fetcher = fetch) {
  return fetcher(`/users/${id}`);
}

// Test with mock
getUser(1, mockFetch);
```

## Summary

Isolate side effects at system edges. Keep core logic pure. Use dependency injection for testability.

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*