---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-fastify-platform"
---

# Fastify Adapter

## Introduction

Express is battle-tested but not the fastest. Fastify can handle significantly more requests per second with lower latency. NestJS's platform-agnostic design lets you switch from Express to Fastify for performance-critical applications.

## Key Concepts

- **Fastify**: High-performance web framework for Node.js
- **FastifyAdapter**: NestJS adapter for Fastify platform
- **JSON Schema**: Fastify uses JSON Schema for validation (optional)
- **Plugins**: Fastify's extension mechanism (instead of middleware)

## Real World Context

Benchmark-dependent, but Fastify often handles 2x the requests per second compared to Express. For high-traffic APIs, this can mean significant cost savings and improved user experience.

## Deep Dive

### Installation

```bash
npm install @nestjs/platform-fastify
```

### Switching to Fastify

```typescript
import { NestFactory } from '@nestjs/core';
import { FastifyAdapter, NestFastifyApplication } from '@nestjs/platform-fastify';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create<NestFastifyApplication>(
    AppModule,
    new FastifyAdapter(),
  );
  
  // IMPORTANT: Fastify requires explicit host for Docker/K8s
  await app.listen(3000, '0.0.0.0');
}
bootstrap();
```

### Fastify Options

```typescript
new FastifyAdapter({
  logger: true,
  trustProxy: true,
  bodyLimit: 10485760, // 10MB
})
```

### Using Fastify Plugins

```typescript
async function bootstrap() {
  const app = await NestFactory.create<NestFastifyApplication>(
    AppModule,
    new FastifyAdapter(),
  );
  
  await app.register(fastifyHelmet);
  await app.register(fastifyCompress);
  await app.register(fastifyCors);
  
  await app.listen(3000, '0.0.0.0');
}
```

### Accessing Fastify Instance

```typescript
const fastifyInstance = app.getHttpAdapter().getInstance();
fastifyInstance.addHook('onRequest', (request, reply, done) => {
  console.log('Request:', request.url);
  done();
});
```

### Migration Considerations

| Express | Fastify |
|---------|---------|
| `app.use(middleware)` | `app.register(plugin)` |
| `req.body` | Same |
| `res.send()` | Same |
| Helmet middleware | @fastify/helmet plugin |

### NestJS 11 Platform Changes

NestJS 11 requires **Node.js v20+** and ships with **Fastify v5** and **Express v5** support.

**Middleware wildcards** changed in both platforms:

```typescript
// Fastify v5: named wildcard
consumer.apply(LoggerMiddleware).forRoutes('*splat');

// Express v5: named wildcard with braces
consumer.apply(LoggerMiddleware).forRoutes('{*splat}');

// Previously (NestJS 10): .forRoutes('(.*)');
```

**Fastify v5 CORS**: By default, only CORS-safelisted methods (GET, HEAD, POST) are allowed. You must explicitly configure methods like PUT, PATCH, and DELETE:

```typescript
const app = await NestFactory.create<NestFastifyApplication>(
  AppModule,
  new FastifyAdapter(),
);
app.enableCors({
  origin: 'https://your-frontend.com',
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
});
```

## Common Pitfalls

1. **Express middleware incompatibility**: Most Express middleware won't work with Fastify. Use Fastify alternatives.
2. **Forgetting '0.0.0.0'**: Fastify binds to localhost by default—breaks containers.
3. **@Res() decorator**: Using @Res() directly bypasses Fastify's optimizations.
4. **Fastify v5 CORS**: Only safelisted HTTP methods are allowed by default — configure explicitly.

## Best Practices

- Test thoroughly after migration
- Use Fastify plugins instead of Express middleware
- Avoid @Res() when possible—let NestJS handle responses
- Profile actual endpoints, not just synthetic benchmarks

## Summary

Fastify provides significant performance improvements over Express. Use FastifyAdapter for the switch, register Fastify plugins instead of Express middleware, and remember to bind to 0.0.0.0 for containerized deployments.

## Code Examples

**Bootstrapping a NestJS app with FastifyAdapter — pass it as the second argument to NestFactory.create()**

```typescript
import { NestFactory } from '@nestjs/core';
import {
  FastifyAdapter,
  NestFastifyApplication,
} from '@nestjs/platform-fastify';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create<NestFastifyApplication>(
    AppModule,
    new FastifyAdapter(),
  );
  // Fastify requires explicit host for Docker/K8s
  await app.listen(3000, '0.0.0.0');
}
bootstrap();
```


## Resources

- [Performance (Fastify)](https://docs.nestjs.com/techniques/performance) — Official Fastify guide

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*