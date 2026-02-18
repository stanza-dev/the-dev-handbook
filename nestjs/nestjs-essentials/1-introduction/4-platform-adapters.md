---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-platform-adapters"
---

# Platform Adapters

## Introduction

One of NestJS's most powerful features is its **platform-agnostic** design. While Express is the default HTTP server, you can switch to Fastify for better performance or even build applications that don't use HTTP at all (microservices, WebSockets, GraphQL).

## Key Concepts

- **Platform Adapter**: The underlying HTTP framework (Express or Fastify)
- **Express**: The default adapter, mature ecosystem, wide middleware compatibility
- **Fastify**: High-performance alternative, ~30k req/sec vs ~15k for Express
- **Abstraction Layer**: NestJS decorators work identically regardless of platform

## Real World Context

Choosing the right platform depends on your needs:
- **Express**: Maximum compatibility with existing npm middleware
- **Fastify**: Raw performance critical, willing to adapt some middleware

Many teams start with Express for familiarity, then migrate to Fastify when performance becomes a bottleneck.

## Deep Dive

### Default Express Setup

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

### Switching to Fastify

```bash
npm install @nestjs/platform-fastify
```

```typescript
import { NestFactory } from '@nestjs/core';
import { FastifyAdapter, NestFastifyApplication } from '@nestjs/platform-fastify';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create<NestFastifyApplication>(
    AppModule,
    new FastifyAdapter()
  );
  await app.listen(3000, '0.0.0.0'); // Fastify requires explicit host
}
bootstrap();
```

### Accessing the Underlying Instance

Sometimes you need platform-specific features:

```typescript
// Express
const expressApp = app.getHttpAdapter().getInstance();

// Fastify
const fastifyInstance = app.getHttpAdapter().getInstance();
```

### Performance Comparison

| Platform | Requests/sec (approximate) | Ecosystem |
|----------|---------------------------|----------|
| Express | ~15,000 | Massive (50k+ middleware packages) |
| Fastify | ~30,000 | Growing (most common needs covered) |

## Common Pitfalls

1. **Express middleware on Fastify**: Most Express middleware won't work on Fastify. Check for Fastify-specific alternatives.
2. **Forgetting `0.0.0.0` with Fastify**: Fastify binds to localhost by default, breaking Docker deployments. Use `app.listen(3000, '0.0.0.0')`.
3. **Using `@Req()` and `@Res()` directly**: These expose platform-specific objects. Prefer `@Body()`, `@Param()`, `@Query()` for portability.

## Best Practices

- **Start with Express**: Unless you have proven performance needs
- **Avoid platform-specific code**: Stick to NestJS decorators for portability
- **Benchmark before switching**: Profile your actual endpoints, not synthetic benchmarks
- **Test thoroughly after migration**: Some edge cases behave differently

## Summary

NestJS abstracts the HTTP layer through platform adapters. Express is the default with maximum ecosystem compatibility, while Fastify offers superior performance. The framework's decorator-based approach means your business logic remains unchanged when switching platforms.

## Resources

- [Performance (Fastify)](https://docs.nestjs.com/techniques/performance) — Guide to using Fastify adapter for improved performance
- [First Steps](https://docs.nestjs.com/first-steps) — Understanding application bootstrapping

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*