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

### URI Versioning (Most Common)

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

### Header Versioning

```typescript
app.enableVersioning({
  type: VersioningType.HEADER,
  header: 'X-API-Version',
});

// Client request:
// GET /users
// X-API-Version: 2
```

### Method-Level Versioning

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

### Multiple Versions on One Method

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

### Version-Neutral Endpoints

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

NestJS versioning supports URI, header, media type, and query strategies. Enable with app.enableVersioning(), apply with @Version() or controller options. URI versioning is most common for public APIs.

## Resources

- [Versioning](https://docs.nestjs.com/techniques/versioning) — Official versioning guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*