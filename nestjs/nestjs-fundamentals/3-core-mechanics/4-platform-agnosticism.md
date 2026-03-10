---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-platform-agnosticism"
---

# Platform Agnosticism

## Introduction

NestJS isn't tied to any specific HTTP framework. It's an abstraction layer that works with different HTTP frameworks and transports. Understanding this architecture helps you write portable code and choose the right platform for your needs.

## Key Concepts

- **HTTP Adapter**: Abstraction over Express/Fastify
- **Platform Package**: @nestjs/platform-express or @nestjs/platform-fastify
- **Transport Layer**: HTTP, WebSocket, microservices, GraphQL

## Real World Context

Teams often start with Express for its massive ecosystem and later need to switch to Fastify for performance, or support both HTTP and WebSocket transports. NestJS's platform abstraction makes these transitions possible without rewriting business logic — a migration that would take weeks in a raw Express app takes hours with NestJS.

## Deep Dive

### Express (Default)

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
```

### Fastify

```typescript
import { NestFactory } from '@nestjs/core';
import { FastifyAdapter, NestFastifyApplication } from '@nestjs/platform-fastify';

async function bootstrap() {
  const app = await NestFactory.create<NestFastifyApplication>(
    AppModule,
    new FastifyAdapter(),
  );
  await app.listen(3000, '0.0.0.0');
}
```

### Accessing Underlying Instance

```typescript
// Express
const expressApp = app.getHttpAdapter().getInstance();
expressApp.use(someExpressMiddleware);

// Fastify
const fastifyInstance = app.getHttpAdapter().getInstance();
await fastifyInstance.register(someFastifyPlugin);
```

### Writing Portable Code

```typescript
// Avoid this (Express-specific)
@Get()
findAll(@Req() req: Request) {
  return req.headers['x-custom'];
}

// Prefer this (portable)
@Get()
findAll(@Headers('x-custom') customHeader: string) {
  return customHeader;
}
```

### Hybrid Applications

```typescript
const app = await NestFactory.create(AppModule);

// Add microservice transport
app.connectMicroservice({
  transport: Transport.REDIS,
  options: { host: 'localhost', port: 6379 },
});

// Add WebSocket gateway
// (WebSocket module handles this automatically)

await app.startAllMicroservices();
await app.listen(3000);
```

### NestJS 11.1: Express v5 & Fastify v5

NestJS 11 upgraded to **Express v5** and **Fastify v5** by default. Key changes:

**Express v5 breaking changes:**
- Wildcard routes require named parameters: `@Get('files/*splat')` instead of `@Get('files/*')`
- Query parser defaults to `simple` — use `app.set('query parser', 'extended')` for nested objects

**Fastify v5** is mostly backward-compatible. Path matching for routes remains unchanged. When using NestJS's middleware abstraction layer (`MiddlewareConsumer.apply()`), path matching follows the updated `path-to-regexp` conventions regardless of the underlying platform. Note that Fastify uses its own native hook system, not Express-style middleware.

**Node.js v20+** is required — support for v16 and v18 has been dropped.

The portable coding patterns shown above remain the best approach regardless of platform version.

## Common Pitfalls

1. **Platform-specific middleware**: Express middleware doesn't work with Fastify.
2. **@Req() and @Res() dependency**: These bind you to a specific platform.
3. **Assuming Express**: Always check documentation for Fastify differences.

## Best Practices

- Use NestJS decorators (@Body, @Query, @Headers) over @Req/@Res
- Test platform changes thoroughly
- Abstract platform-specific code into services
- Document platform requirements

## Summary

NestJS abstracts the HTTP layer through adapters. Express is the default; Fastify offers better performance. Write portable code using NestJS decorators instead of accessing raw request/response objects. Hybrid applications can combine HTTP with WebSockets and microservices.

## Code Examples

**Platform-agnostic code — use @Param(), @Headers(), @Query() instead of @Req() to keep your code portable across Express and Fastify**

```typescript
// Portable code using NestJS decorators
@Get(':id')
findOne(
  @Param('id') id: string,
  @Headers('x-tenant-id') tenantId: string,
  @Query('include') include?: string,
) {
  return this.service.findOne(id, tenantId, include);
}

// Avoid: @Req() req — ties you to Express/Fastify
```


## Resources

- [Performance (Fastify)](https://docs.nestjs.com/techniques/performance) — Fastify integration guide

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*