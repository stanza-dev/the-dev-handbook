---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-platform-agnosticism"
---

# Platform Agnosticism

## Introduction

NestJS isn't tied to Express. It's an abstraction layer that works with different HTTP frameworks and transports. Understanding this architecture helps you write portable code and choose the right platform for your needs.

## Key Concepts

- **HTTP Adapter**: Abstraction over Express/Fastify
- **Platform Package**: @nestjs/platform-express or @nestjs/platform-fastify
- **Transport Layer**: HTTP, WebSocket, microservices, GraphQL

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

## Resources

- [Performance (Fastify)](https://docs.nestjs.com/techniques/performance) — Fastify integration guide

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*