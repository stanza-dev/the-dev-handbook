---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-proxy-pattern"
---

# The Proxy Pattern

## Introduction

The Proxy pattern provides a surrogate for another object to control access. JavaScript's native Proxy API enables powerful metaprogramming.

## Deep Dive

### Basic Proxy

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

### Validation Proxy

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

### Reactive Proxy

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

## Summary

Proxies intercept object operations through handler traps. Use for validation, logging, reactivity, and access control. Consider performance implications.

## Resources

- [MDN: Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) — Proxy reference

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*