---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-breaking-changes"
---

# Managing Breaking Changes

## Introduction

Breaking changes are sometimes necessary for API improvement. The key is managing them without breaking existing clients. Proper communication, deprecation periods, and migration support make transitions smooth.

## Key Concepts

- **Breaking Change**: Change that breaks existing clients
- **Backward Compatibility**: New version works with old clients
- **Forward Compatibility**: Old version works with new clients
- **Graceful Degradation**: Handle missing fields gracefully

## Real World Context

Breaking changes include:
- Removing fields or endpoints
- Changing field types
- Renaming fields
- Modifying authentication
- Changing response format

## Deep Dive

### Non-Breaking vs Breaking

Understanding the difference between additive and destructive changes is fundamental to versioning decisions.

```typescript
// NON-BREAKING: Adding optional field
{
  id: '123',
  name: 'John',
  email: 'john@example.com',  // New field - old clients ignore it
}

// BREAKING: Removing field
{
  id: '123',
  // name removed - old clients fail
}

// BREAKING: Changing type
{
  id: 123,  // Was string, now number
}

// BREAKING: Renaming field
{
  userId: '123',  // Was 'id' - old clients fail
}
```

Additive changes (new optional fields, new endpoints) are safe; removals, renames, and type changes break existing clients.

### Supporting Both Formats

Use `@Transform()` to compute the old field from the new fields, keeping backward compatibility during the transition.

```typescript
export class UserV2Dto {
  @ApiProperty()
  id: string;

  @ApiProperty()
  firstName: string;

  @ApiProperty()
  lastName: string;

  // Keep old field for backward compatibility
  @ApiProperty({ deprecated: true })
  @Transform(({ obj }) => `${obj.firstName} ${obj.lastName}`)
  name: string;
}
```

Marking the old field with `deprecated: true` signals to API consumers that they should migrate to the new fields.

### Deprecation Decorator

Build a reusable `@Deprecated()` decorator that marks the endpoint in Swagger and adds Sunset headers.

```typescript
export function Deprecated(message: string, sunsetDate: string) {
  return applyDecorators(
    ApiOperation({ deprecated: true }),
    UseInterceptors(
      new (class implements NestInterceptor {
        intercept(context: ExecutionContext, next: CallHandler) {
          const response = context.switchToHttp().getResponse();
          response.setHeader('Deprecation', 'true');
          response.setHeader('Sunset', sunsetDate);
          response.setHeader('X-Deprecation-Message', message);
          return next.handle();
        }
      })(),
    ),
  );
}

// Usage
@Get('legacy')
@Deprecated('Use GET /v2/users instead', 'Sat, 31 Dec 2025 23:59:59 GMT')
findAllLegacy() { ... }
```

The `X-Deprecation-Message` custom header gives clients a human-readable migration hint.

### Field Aliasing for Migration

Accept both old and new field names during the transition period using a `@Transform()` that maps the legacy field.

```typescript
export class CreateUserDto {
  // Accept both old and new field names
  @ApiPropertyOptional()
  @IsOptional()
  @IsString()
  firstName?: string;

  @ApiPropertyOptional({ deprecated: true })
  @IsOptional()
  @IsString()
  @Transform(({ value, obj }) => {
    // If name provided but firstName not, use name
    if (value && !obj.firstName) {
      const [first, ...rest] = value.split(' ');
      obj.firstName = first;
      obj.lastName = rest.join(' ');
    }
    return value;
  })
  name?: string;  // Old field still accepted
}
```

The transform splits the legacy `name` into `firstName` and `lastName` only when the new fields are not provided.

### Version Comparison Middleware

Create an interceptor that warns clients using outdated versions and rejects unsupported ones.

```typescript
@Injectable()
export class VersionWarningInterceptor implements NestInterceptor {
  private readonly latestVersion = '2';
  private readonly supportedVersions = ['1', '2'];

  intercept(context: ExecutionContext, next: CallHandler) {
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();
    const version = request.version || '1';

    if (!this.supportedVersions.includes(version)) {
      throw new BadRequestException(`Version ${version} not supported`);
    }

    if (version !== this.latestVersion) {
      response.setHeader('X-API-Warn', `You are using v${version}. Latest is v${this.latestVersion}`);
    }

    return next.handle();
  }
}
```

The `X-API-Warn` header is non-intrusive but gives client developers a visible nudge to upgrade.

### Changelog Communication

Expose an API-consumable changelog endpoint that lists version changes, types, and migration links.

```typescript
@Controller('changelog')
export class ChangelogController {
  @Get()
  @ApiOperation({ summary: 'Get API changelog' })
  getChangelog() {
    return {
      versions: [
        {
          version: '2.0',
          date: '2025-01-01',
          changes: [
            { type: 'breaking', description: 'User name split into firstName/lastName' },
            { type: 'added', description: 'Pagination support' },
          ],
          migration: 'https://docs.example.com/migrate-v2',
        },
        {
          version: '1.1',
          date: '2024-06-01',
          changes: [
            { type: 'added', description: 'Email field added to users' },
          ],
        },
      ],
    };
  }
}
```

Including a `migration` URL for each breaking version gives developers a direct path to upgrade instructions.

## Common Pitfalls

1. **Surprise breaking changes**: Always communicate in advance.
2. **Short deprecation windows**: Give clients time to migrate.
3. **No migration guide**: Document how to upgrade.

## Best Practices

- Announce breaking changes well in advance
- Provide migration guides and code examples
- Support both old and new formats during transition
- Use Sunset headers to communicate end-of-life
- Monitor old version usage before removing

## Summary

- Breaking changes include removing fields, changing types, and renaming fields
- Support both old and new formats during the transition period
- Use Sunset and Deprecation headers to communicate end-of-life dates
- Provide migration guides with code examples for each breaking change
- Monitor old version usage and announce changes well in advance of removal

## Code Examples

**Comparing non-breaking vs breaking response changes with concrete examples**

```typescript
// NON-BREAKING: Adding optional field
const responseV1 = {
  id: '123',
  name: 'John',
  email: 'john@example.com',  // New field - old clients ignore it
};

// BREAKING: Removing field
const responseV2Bad = {
  id: '123',
  // name removed - old clients fail
};

// BREAKING: Changing type
const responseV3Bad = {
  id: 123,  // Was string, now number
};

// BREAKING: Renaming field
const responseV4Bad = {
  userId: '123',  // Was 'id' - old clients fail
};
```


## Resources

- [Versioning](https://docs.nestjs.com/techniques/versioning) — Version management

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*