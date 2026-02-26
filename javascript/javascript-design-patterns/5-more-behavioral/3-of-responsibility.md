---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-chain-of-responsibility"
---

# Chain of Responsibility Pattern

## Introduction

The Chain of Responsibility pattern passes a request along a chain of handlers. Each handler decides to process or pass to the next handler.

## Key Concepts

**Handler**: A processor that can handle a request or pass it to the next handler.

**Chain**: The linked sequence of handlers.

**Next**: The mechanism to pass a request to the next handler in the chain.

**Short-Circuiting**: A handler can stop the chain by not calling next.

## Real World Context

Express/Koa middleware stacks are the most common web example. DOM event bubbling, error handling pipelines, and approval workflows (manager → director → VP) all implement this pattern. Webpack loaders process files through a chain of transformers.

## Deep Dive

### Middleware Chain

Each handler in the chain processes the request if it can, or delegates to the next handler via `super.handle()`. Authentication and rate limiting are checked before processing:

```javascript
class Handler {
  setNext(handler) {
    this.next = handler;
    return handler;
  }
  
  handle(request) {
    if (this.next) return this.next.handle(request);
    return null;
  }
}

class AuthHandler extends Handler {
  handle(request) {
    if (!request.user) {
      return { error: 'Unauthorized' };
    }
    return super.handle(request);
  }
}

class RateLimitHandler extends Handler {
  #requests = new Map();
  
  handle(request) {
    const count = this.#requests.get(request.user) || 0;
    if (count > 100) {
      return { error: 'Rate limited' };
    }
    this.#requests.set(request.user, count + 1);
    return super.handle(request);
  }
}

class ProcessHandler extends Handler {
  handle(request) {
    return { success: true, data: process(request) };
  }
}

const chain = new AuthHandler();
chain.setNext(new RateLimitHandler())
     .setNext(new ProcessHandler());

chain.handle({ user: 'alice', data: {} });
```

The `setNext()` method returns the next handler, enabling fluent chaining: `chain.setNext(b).setNext(c)`. If `AuthHandler` rejects the request, the chain short-circuits and `ProcessHandler` never runs.

### Express-style Middleware

This functional version mirrors Express middleware. Each handler receives a `next` callback that advances to the next handler in the pipeline:

```javascript
function createPipeline(...handlers) {
  return (request) => {
    let index = 0;
    function next() {
      const handler = handlers[index++];
      if (handler) handler(request, next);
    }
    next();
  };
}

const pipeline = createPipeline(
  (req, next) => { console.log('Logging'); next(); },
  (req, next) => { req.parsed = true; next(); },
  (req) => { console.log('Done', req); }
);
```

The last handler does not call `next()`, terminating the chain. If any middle handler omits `next()`, the remaining handlers are silently skipped.

## Common Pitfalls

1. **Broken chain** — Forgetting to call `next()` silently drops the request. Always call `next()` or explicitly return a response.
2. **Order dependency** — Handler order matters. Authentication must come before authorization. Document the expected order.
3. **Too many handlers** — Long chains are hard to debug. Keep chains under 5-7 handlers and consider grouping related handlers.

## Best Practices

1. **Make handlers independent** — Each handler should work without knowledge of which handlers come before or after.
2. **Use a fluent API for chain construction** — `.setNext(handler)` returning the handler enables chaining: `a.setNext(b).setNext(c)`.
3. **Log which handler processed the request** — In debugging, knowing where in the chain a request was handled is invaluable.

## Summary

Chain of Responsibility passes requests through handlers. Each handler can process or pass along. Use for middleware, validation chains, event handling.

## Code Examples

**Express-style middleware pipeline — each handler calls next() to pass the request to the next handler in the chain**

```javascript
function createPipeline(...handlers) {
  return (request) => {
    let index = 0;
    function next() {
      const handler = handlers[index++];
      if (handler) handler(request, next);
    }
    next();
  };
}

const pipeline = createPipeline(
  (req, next) => { console.log('Auth check'); next(); },
  (req, next) => { console.log('Rate limit'); next(); },
  (req) => { console.log('Process request'); }
);

pipeline({ user: 'alice' });
// Auth check → Rate limit → Process request
```


## Resources

- [Refactoring Guru: Chain of Responsibility](https://refactoring.guru/design-patterns/chain-of-responsibility) — Chain of Responsibility guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*