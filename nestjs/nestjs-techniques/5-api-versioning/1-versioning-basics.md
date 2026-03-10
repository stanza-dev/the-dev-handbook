---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-versioning-basics"
---

# API Versioning Fundamentals

## Introduction

APIs evolve. Breaking changes are sometimes necessary. Versioning allows you to introduce changes while maintaining backward compatibility for existing clients. NestJS provides built-in versioning support.

## Key Concepts

- **URI Versioning**: Version in URL path (/v1/users)
- **Header Versioning**: Version in custom header
- **Media Type Versioning**: Version in Accept header
- **Query Versioning**: Version in query parameter

## Real World Context

Versioning scenarios:
- Changing response format
- Removing deprecated fields
- Modifying authentication flow
- Adding required parameters

## Deep Dive

### Enabling Versioning

Call `app.enableVersioning()` in your bootstrap function and choose a versioning strategy.

```typescript
import { VersioningType } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.enableVersioning({
    type: VersioningType.URI,  // /v1/users, /v2/users
    defaultVersion: '1',
  });

  await app.listen(3000);
}
```

The `defaultVersion` handles requests that do not specify a version explicitly.

### URI Versioning (Most Common)

With URI versioning, the version is embedded in the URL path. Each controller declares which version it handles.

```typescript
app.enableVersioning({
  type: VersioningType.URI,
  prefix: 'v', // Results in /v1/, /v2/
});

@Controller({
  path: 'users',
  version: '1',
})
export class UsersV1Controller {
  @Get()
  findAll() {
    return 'V1 Users';
  }
}

@Controller({
  path: 'users',
  version: '2',
})
export class UsersV2Controller {
  @Get()
  findAll() {
    return 'V2 Users with new format';
  }
}
```

Both controllers share the same `path` but are differentiated by the `version` field in the decorator options.

### Header Versioning

Header versioning keeps URLs clean by sending the version in a custom HTTP header.

```typescript
app.enableVersioning({
  type: VersioningType.HEADER,
  header: 'X-API-Version',
});

// Client request:
// GET /users
// X-API-Version: 2
```

This approach is useful for internal APIs where URL aesthetics matter more than browser testability.

### Method-Level Versioning

Apply the `@Version()` decorator to individual methods instead of the entire controller for finer control.

```typescript
@Controller('users')
export class UsersController {
  @Get()
  @Version('1')
  findAllV1() {
    return 'V1 format';
  }

  @Get()
  @Version('2')
  findAllV2() {
    return 'V2 format';
  }
}
```

This is convenient when only a few methods differ between versions and most logic is shared.

### Multiple Versions on One Method

Pass an array to `@Version()` to serve the same handler for multiple versions.

```typescript
@Controller('users')
export class UsersController {
  @Get()
  @Version(['1', '2'])
  findAll() {
    return 'Works for both V1 and V2';
  }
}
```

This avoids duplicating handlers when the response format has not changed between versions.

### Version-Neutral Endpoints

Use `VERSION_NEUTRAL` for endpoints like health checks that should respond regardless of the requested version.

```typescript
@Controller({
  path: 'health',
  version: VERSION_NEUTRAL,
})
export class HealthController {
  @Get()
  check() {
    return { status: 'ok' };
  }
}
```

Neutral endpoints are accessible at both `/health` and `/v1/health`, making them ideal for monitoring probes.

### Versioning Types Comparison

| Type | Example | Pros | Cons |
|------|---------|------|------|
| URI | /v1/users | Clear, cacheable | URL changes |
| Header | X-API-Version: 1 | Clean URLs | Harder to test |
| Media Type | Accept: application/vnd.api.v1+json | RESTful | Complex |
| Query | ?version=1 | Easy to test | Pollutes query |

## Common Pitfalls

1. **No default version**: Always set a default for versionless requests.
2. **Too many versions**: Maintain only 2-3 active versions.
3. **Breaking changes in minor versions**: Follow semantic versioning.

## Best Practices

- Use URI versioning for public APIs (most visible)
- Use header versioning for internal APIs
- Set a clear deprecation policy
- Document version differences
- Support at least N-1 version

## Summary

- NestJS supports URI, header, media type, and query versioning strategies
- Enable versioning globally with app.enableVersioning() and set a defaultVersion
- Apply versions at controller level or per-method with the @Version() decorator
- Use VERSION_NEUTRAL for endpoints like health checks that don't change between versions
- URI versioning is most common for public APIs due to visibility and cacheability

## Code Examples

**Enabling URI versioning in a NestJS application with app.enableVersioning()**

```typescript
import { VersioningType } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.enableVersioning({
    type: VersioningType.URI,  // /v1/users, /v2/users
    defaultVersion: '1',
  });

  await app.listen(3000);
}
```


## Resources

- [Versioning](https://docs.nestjs.com/techniques/versioning) — Official versioning guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*