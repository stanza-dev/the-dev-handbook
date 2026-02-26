---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-proxy-pattern"
---

# The Proxy Pattern

## Introduction

The Proxy pattern provides a surrogate for another object to control access. JavaScript's native Proxy API enables powerful metaprogramming.

## Key Concepts

**Proxy**: A surrogate object that controls access to another object.

**Handler/Traps**: Methods on the handler that intercept operations (get, set, apply, etc.).

**Target**: The original object the Proxy wraps.

**Revocable Proxy**: A proxy that can be permanently disabled via `Proxy.revocable()`.

## Real World Context

Vue 3's reactivity system is built on Proxy. Validation layers, access control, lazy loading, and change tracking all use Proxy traps. MobX uses Proxy to make state observable automatically.

## Deep Dive

### Basic Proxy

The `get` and `set` traps intercept every property access and assignment on the proxied object, letting us add logging transparently:

```javascript
const user = { name: 'Alice', age: 25 };

const proxy = new Proxy(user, {
  get(target, prop) {
    console.log(`Accessing ${prop}`);
    return target[prop];
  },
  set(target, prop, value) {
    console.log(`Setting ${prop} = ${value}`);
    target[prop] = value;
    return true;
  }
});

proxy.name;  // Logs 'Accessing name'
```

The proxy looks and behaves exactly like the original `user` object. Code consuming the proxy does not need to know it is wrapped.

### Validation Proxy

A validation proxy enforces type constraints inside the `set` trap, throwing errors for invalid assignments before they reach the target:

```javascript
const validator = {
  set(target, prop, value) {
    if (prop === 'age' && typeof value !== 'number') {
      throw new TypeError('Age must be a number');
    }
    target[prop] = value;
    return true;
  }
};

const user = new Proxy({}, validator);
user.age = 'invalid';  // TypeError!
```

The `set` trap must return `true` to indicate success. Returning `false` or forgetting the return causes a `TypeError` in strict mode.

### Reactive Proxy

Reactive proxies power modern frameworks like Vue 3. By hooking into the `set` trap, the proxy can trigger re-renders whenever state changes:

```javascript
function reactive(obj, onChange) {
  return new Proxy(obj, {
    set(target, prop, value) {
      target[prop] = value;
      onChange(prop, value);
      return true;
    }
  });
}

const state = reactive({ count: 0 }, (p, v) => {
  console.log(`${p} changed to ${v}`);
});
state.count++;  // 'count changed to 1'
```

The `onChange` callback fires on every assignment, making this a simple but effective reactivity system. This is the core idea behind Vue 3's `reactive()` function.

## Common Pitfalls

1. **Performance overhead** — Every property access goes through the trap. Avoid proxying hot-path objects accessed thousands of times per frame.
2. **Missing `return true` in `set`** — The `set` trap must return `true` or a TypeError is thrown in strict mode.
3. **Proxy identity** — `proxy !== target`, which can break `===` checks and Map/Set lookups.

## Best Practices

1. **Use `Reflect` methods inside traps** — `Reflect.get(target, prop, receiver)` preserves correct `this` binding and default behavior.
2. **Validate at boundaries only** — Wrap objects in validation proxies at API boundaries, not deep inside hot loops.
3. **Use `Proxy.revocable()` for temporary access** — Revocable proxies let you cut off access when a session or permission expires.
4. **Combine with ES2025 `using` for automatic cleanup** — Proxies can implement `Symbol.dispose` so resources are released when a block exits:

```javascript
function createProxiedResource(target) {
  const { proxy, revoke } = Proxy.revocable(target, {});
  proxy[Symbol.dispose] = () => revoke();
  return proxy;
}
{
  using handle = createProxiedResource({ data: 42 });
  handle.data; // 42
} // handle[Symbol.dispose]() called — proxy revoked automatically
```

The `using` keyword (ES2025 Explicit Resource Management) calls `[Symbol.dispose]()` when the block exits, pairing naturally with revocable proxies.

## Summary

Proxies intercept object operations through handler traps. Use for validation, logging, reactivity, and access control. Consider performance implications.

## Code Examples

**Validation Proxy — the set trap intercepts property assignment to enforce type and value constraints**

```javascript
const validator = {
  set(target, prop, value) {
    if (prop === 'age' && (typeof value !== 'number' || value < 0)) {
      throw new TypeError('Age must be a non-negative number');
    }
    if (prop === 'email' && !value.includes('@')) {
      throw new TypeError('Invalid email address');
    }
    target[prop] = value;
    return true;
  }
};

const user = new Proxy({}, validator);
user.name = 'Alice';  // OK
user.age = 25;        // OK
user.age = -1;        // TypeError: Age must be a non-negative number
```


## Resources

- [MDN: Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) — Proxy reference

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*