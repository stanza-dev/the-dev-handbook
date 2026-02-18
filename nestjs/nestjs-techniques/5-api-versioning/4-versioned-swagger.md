---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-versioned-swagger"
---

# Versioned API Documentation

## Introduction

When your API has multiple versions, documentation should reflect each version's specifics. Swagger can generate separate documentation for each version, helping developers understand what's available in each.

## Key Concepts

- **Multi-Document Setup**: Separate Swagger docs per version
- **Version Tags**: Group endpoints by version
- **Deprecation Markers**: Show deprecated endpoints
- **Version Comparison**: Help clients see differences

## Real World Context

Versioned documentation helps:
- Developers using specific versions
- Migration planning
- API exploration
- Support team troubleshooting

## Deep Dive

### Multiple Swagger Documents

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.enableVersioning({
    type: VersioningType.URI,
  });

  // V1 Documentation
  const v1Config = new DocumentBuilder()
    .setTitle('API v1')
    .setDescription('Version 1 of the API (Deprecated)')
    .setVersion('1.0')
    .addBearerAuth()
    .build();
  const v1Document = SwaggerModule.createDocument(app, v1Config, {
    include: [UsersV1Module, ProductsV1Module],
  });
  SwaggerModule.setup('api/v1/docs', app, v1Document);

  // V2 Documentation
  const v2Config = new DocumentBuilder()
    .setTitle('API v2')
    .setDescription('Version 2 of the API (Current)')
    .setVersion('2.0')
    .addBearerAuth()
    .build();
  const v2Document = SwaggerModule.createDocument(app, v2Config, {
    include: [UsersV2Module, ProductsV2Module],
  });
  SwaggerModule.setup('api/v2/docs', app, v2Document);

  await app.listen(3000);
}
```

### Version Tags in Single Document

```typescript
const config = new DocumentBuilder()
  .setTitle('API')
  .addTag('v1-users', 'Users API v1 (deprecated)')
  .addTag('v2-users', 'Users API v2')
  .build();

@Controller({ path: 'users', version: '1' })
@ApiTags('v1-users')
export class UsersV1Controller { ... }

@Controller({ path: 'users', version: '2' })
@ApiTags('v2-users')
export class UsersV2Controller { ... }
```

### Deprecation in Swagger

```typescript
@Controller('users')
export class UsersController {
  @Get()
  @Version('1')
  @ApiOperation({
    summary: 'Get all users',
    deprecated: true,
    description: 'DEPRECATED: Use /v2/users instead. Will be removed on 2025-12-31.',
  })
  findAllV1() { ... }

  @Get()
  @Version('2')
  @ApiOperation({ summary: 'Get all users with pagination' })
  findAllV2() { ... }
}
```

### Schema Versioning

```typescript
// Register both DTO versions
const document = SwaggerModule.createDocument(app, config, {
  extraModels: [UserV1Dto, UserV2Dto],
});

// Reference specific version
@Get()
@Version('1')
@ApiOkResponse({ type: UserV1Dto })
findAllV1() { ... }

@Get()
@Version('2')
@ApiOkResponse({ type: UserV2Dto })
findAllV2() { ... }
```

### Documentation Landing Page

```typescript
@Controller()
export class ApiDocsController {
  @Get()
  @ApiExcludeEndpoint()
  redirectToDocs(@Res() res: Response) {
    res.redirect('/api/v2/docs');
  }
}

// Or create a landing page
@Controller('api')
export class ApiVersionsController {
  @Get()
  getVersions() {
    return {
      versions: [
        {
          version: 'v1',
          status: 'deprecated',
          docsUrl: '/api/v1/docs',
          sunsetDate: '2025-12-31',
        },
        {
          version: 'v2',
          status: 'current',
          docsUrl: '/api/v2/docs',
        },
      ],
      latest: 'v2',
    };
  }
}
```

### Version-Specific Examples

```typescript
export class UserV1Dto {
  @ApiProperty({ example: '123' })
  id: string;

  @ApiProperty({ example: 'John Doe' })
  name: string;
}

export class UserV2Dto {
  @ApiProperty({ example: '123' })
  id: string;

  @ApiProperty({ example: 'John' })
  firstName: string;

  @ApiProperty({ example: 'Doe' })
  lastName: string;
}
```

## Common Pitfalls

1. **Single doc for all versions**: Confusing when schemas differ.
2. **No deprecation markers**: Users don't know what's outdated.
3. **Missing examples**: Version differences unclear without them.

## Best Practices

- Create separate Swagger documents per major version
- Mark deprecated endpoints clearly
- Include examples showing version differences
- Provide a landing page listing all versions
- Link to migration guides from deprecated endpoints

## Summary

Versioned Swagger documentation uses multiple documents or tags for different versions. Mark deprecated endpoints, include version-specific examples, and provide a landing page listing available versions with their status.

## Resources

- [OpenAPI](https://docs.nestjs.com/openapi/introduction) — Swagger documentation

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*