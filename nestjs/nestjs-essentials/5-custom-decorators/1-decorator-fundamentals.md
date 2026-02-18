---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-decorator-fundamentals"
---

# Decorator Fundamentals

## Introduction

Decorators are the backbone of NestJS. Every `@Controller()`, `@Injectable()`, and `@Get()` you've used is a decorator. Understanding how to create your own unlocks powerful patterns for reducing boilerplate, enforcing conventions, and building expressive APIs.

## Key Concepts

- **Decorator**: A function that adds metadata or modifies behavior of classes, methods, or parameters
- **Decorator Factory**: A function that returns a decorator (enables passing arguments)
- **Metadata**: Data attached to classes/methods via `Reflect.defineMetadata()`
- **Composition**: Combining multiple decorators into one

## Real World Context

In production NestJS applications, custom decorators:
- Extract user information from requests (`@CurrentUser()`)
- Mark routes as public (`@Public()`)
- Add caching metadata (`@CacheTTL(60)`)
- Combine multiple decorators (`@ApiAuth()` = Guards + Swagger docs)

## Deep Dive

### TypeScript Decorator Types

| Type | Applied To | Signature |
|------|-----------|----------|
| Class | Classes | `(target: Function) => void` |
| Method | Methods | `(target, key, descriptor) => void` |
| Property | Properties | `(target, key) => void` |
| Parameter | Parameters | `(target, key, index) => void` |

### Basic Decorator Structure

```typescript
// Simple decorator (no arguments)
function Log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with`, args);
    return original.apply(this, args);
  };
}

// Decorator factory (with arguments)
function Log(prefix: string) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const original = descriptor.value;
    descriptor.value = function (...args: any[]) {
      console.log(`[${prefix}] Calling ${propertyKey}`);
      return original.apply(this, args);
    };
  };
}
```

### Using Reflect Metadata

```typescript
import 'reflect-metadata';

const ROLES_KEY = 'roles';

function Roles(...roles: string[]) {
  return (target: any, key?: string, descriptor?: PropertyDescriptor) => {
    if (descriptor) {
      Reflect.defineMetadata(ROLES_KEY, roles, descriptor.value);
    } else {
      Reflect.defineMetadata(ROLES_KEY, roles, target);
    }
  };
}

// Reading metadata later
const roles = Reflect.getMetadata(ROLES_KEY, target);
```

### NestJS's SetMetadata Helper

```typescript
import { SetMetadata } from '@nestjs/common';

export const IS_PUBLIC_KEY = 'isPublic';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);

// Usage
@Public()
@Get('health')
health() { return 'OK'; }
```

## Common Pitfalls

1. **Decorator order matters**: Decorators execute bottom-to-top. `@A() @B()` runs B first, then A.
2. **Forgetting to enable metadata**: Ensure `emitDecoratorMetadata` and `experimentalDecorators` are true in `tsconfig.json`.
3. **Arrow functions in method decorators**: Arrow functions don't have `this`. Use regular functions.

## Best Practices

- Use `SetMetadata` for simple key-value metadata in NestJS
- Create decorator factories when you need arguments
- Keep decorators focused on one responsibility
- Document custom decorators for team visibility

## Summary

Decorators are functions that modify classes, methods, properties, or parameters. In NestJS, use `SetMetadata()` for attaching metadata and `Reflector` to read it. Decorator factories allow parameterized decorators. Remember that execution order is bottom-to-top.

## Resources

- [Custom Decorators](https://docs.nestjs.com/custom-decorators) — Official guide to creating custom decorators
- [TypeScript Decorators](https://www.typescriptlang.org/docs/handbook/decorators.html) — TypeScript decorator documentation

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*