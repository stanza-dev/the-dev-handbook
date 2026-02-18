---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-injection-scopes"
---

# Injection Scopes

## Introduction

By default, NestJS providers are singletons—one instance shared across the entire application. But sometimes you need request-specific instances or fresh instances on every injection. Injection scopes control provider lifecycle.

## Key Concepts

- **DEFAULT (Singleton)**: One instance for entire app lifetime
- **REQUEST**: New instance per HTTP request
- **TRANSIENT**: New instance on every injection

## Real World Context

Use cases for different scopes:
- **Singleton**: Stateless services, database connections, caches
- **Request**: Request-specific data (user context, tenant info)
- **Transient**: Stateful helpers that shouldn't share state

## Deep Dive

### Singleton (Default)

```typescript
@Injectable()
export class CatsService {
  // One instance shared across all requests
}
```

### Request Scope

```typescript
import { Injectable, Scope } from '@nestjs/common';

@Injectable({ scope: Scope.REQUEST })
export class RequestContextService {
  private userId: string;
  
  setUserId(id: string) {
    this.userId = id;
  }
  
  getUserId(): string {
    return this.userId;
  }
}
```

### Transient Scope

```typescript
@Injectable({ scope: Scope.TRANSIENT })
export class HelperService {
  private state: any;
  // Fresh instance every time it's injected
}
```

### Scope Bubbling

When a singleton depends on a request-scoped provider, the singleton becomes request-scoped too:

```typescript
@Injectable() // Would become request-scoped
export class ServiceA {
  constructor(private requestScoped: RequestContextService) {}
}
```

### Accessing Request in Request-Scoped Providers

```typescript
import { REQUEST } from '@nestjs/core';
import { Request } from 'express';

@Injectable({ scope: Scope.REQUEST })
export class RequestService {
  constructor(@Inject(REQUEST) private request: Request) {
    console.log(request.url);
  }
}
```

## Common Pitfalls

1. **Scope bubbling surprises**: Injecting request-scoped into singleton makes it request-scoped. Performance impact.
2. **Memory leaks with REQUEST scope**: Many request-scoped providers = many instances. Monitor memory.
3. **Not understanding singleton default**: Storing request data in singletons causes race conditions.

## Best Practices

- Stick with singleton (default) unless you have a specific need
- Use request scope sparingly—it has performance overhead
- For user context, consider a request-scoped context service
- Document non-default scopes for team awareness

## Summary

Provider scopes control instance lifecycle. DEFAULT (singleton) shares one instance, REQUEST creates per-request instances, and TRANSIENT creates fresh instances on every injection. Use the appropriate scope based on your statefulness requirements.

## Resources

- [Injection Scopes](https://docs.nestjs.com/fundamentals/injection-scopes) — Official injection scopes guide

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*