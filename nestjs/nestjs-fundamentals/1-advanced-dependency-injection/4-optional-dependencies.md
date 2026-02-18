---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-optional-dependencies"
---

# Optional Dependencies & Tokens

## Introduction

Not all dependencies are required. Some features are optional, some services might not be registered. NestJS provides mechanisms to handle missing dependencies gracefully rather than crashing on startup.

## Key Concepts

- **@Optional()**: Decorator for optional dependencies
- **Injection Token**: Identifier for dependency lookup (class or string/symbol)
- **Symbol Tokens**: Unique identifiers to avoid string collisions
- **Self/SkipSelf**: Control where dependency is resolved from

## Real World Context

Scenarios for optional dependencies:
- Optional logging service (may or may not be configured)
- Plugin systems where features are pluggable
- Conditional feature flags

## Deep Dive

### Optional Decorator

```typescript
import { Injectable, Optional } from '@nestjs/common';

@Injectable()
export class NotificationService {
  constructor(
    @Optional() private emailService?: EmailService,
    @Optional() private smsService?: SmsService,
  ) {}

  notify(message: string) {
    if (this.emailService) {
      this.emailService.send(message);
    }
    if (this.smsService) {
      this.smsService.send(message);
    }
  }
}
```

### String Tokens

```typescript
// Define token
export const CONFIG_OPTIONS = 'CONFIG_OPTIONS';

// Provider
{
  provide: CONFIG_OPTIONS,
  useValue: { debug: true },
}

// Injection
constructor(@Inject(CONFIG_OPTIONS) private options: ConfigOptions) {}
```

### Symbol Tokens (Recommended)

```typescript
// tokens.ts
export const DATABASE_CONNECTION = Symbol('DATABASE_CONNECTION');

// Provider
{
  provide: DATABASE_CONNECTION,
  useFactory: () => createConnection(),
}

// Injection
constructor(@Inject(DATABASE_CONNECTION) private db: Connection) {}
```

### Self and SkipSelf

```typescript
@Injectable()
export class ChildService {
  constructor(
    @Self() private strictlyLocal: LocalService,      // Must be in same module
    @SkipSelf() private fromParent: ParentService,    // Must be from parent
  ) {}
}
```

### Default Values with Optional

```typescript
@Injectable()
export class AppService {
  private readonly logger: Logger;
  
  constructor(@Optional() logger?: Logger) {
    this.logger = logger ?? new ConsoleLogger();
  }
}
```

## Common Pitfalls

1. **Forgetting null checks**: @Optional() makes dependency undefined, not a mock. Check before using.
2. **String token typos**: Use Symbol or constants to avoid typos in string tokens.
3. **Overusing optional**: If a dependency is always needed, don't make it optional.

## Best Practices

- Use Symbol for custom tokens to avoid collisions
- Create a tokens.ts file for centralized token definitions
- Always provide fallback behavior for optional dependencies
- Document which dependencies are optional and why

## Summary

@Optional() prevents crashes for missing dependencies. Use string or Symbol tokens for non-class providers. Symbol tokens are preferred for uniqueness. Implement fallback behavior when optional dependencies are missing.

## Resources

- [Custom Providers](https://docs.nestjs.com/fundamentals/custom-providers#optional-providers) — Optional providers documentation

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*