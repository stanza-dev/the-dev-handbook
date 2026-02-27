---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-chain-of-responsibility"
---

# Chain of Responsibility Pattern

## Introduction

The Chain of Responsibility pattern passes a request along a chain of handlers. Each handler decides to process or pass to the next handler.

## Deep Dive

### Middleware Chain

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

### Express-style Middleware

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

## Summary

Chain of Responsibility passes requests through handlers. Each handler can process or pass along. Use for middleware, validation chains, event handling.

## Resources

- [Refactoring Guru: Chain of Responsibility](https://refactoring.guru/design-patterns/chain-of-responsibility) — Chain of Responsibility guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*