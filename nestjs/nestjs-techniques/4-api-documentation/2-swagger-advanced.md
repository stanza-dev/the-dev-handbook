---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-swagger-advanced"
---

# Advanced Swagger Features

## Introduction

Beyond basic documentation, Swagger supports authentication flows, multiple API versions, custom schemas, and document generation for different audiences. Master these features for professional-grade API documentation.

## Key Concepts

- **Security Schemes**: Document authentication methods
- **Multiple Documents**: Separate docs for different audiences
- **Extra Models**: Include DTOs not directly referenced
- **Custom Decorators**: Combine multiple Swagger decorators

## Real World Context

Advanced features enable:
- Testing authenticated endpoints in Swagger UI
- Separate docs for public vs internal APIs
- Generated SDKs with proper types
- Custom documentation workflows

## Deep Dive

### Authentication Schemes

Register multiple security schemes (Bearer JWT, API Key, OAuth2) on the `DocumentBuilder` to enable the Authorize button in Swagger UI.

```typescript
const config = new DocumentBuilder()
  .setTitle('API')
  .addBearerAuth(
    {
      type: 'http',
      scheme: 'bearer',
      bearerFormat: 'JWT',
      description: 'Enter JWT token',
    },
    'JWT-auth', // Name for reference
  )
  .addApiKey(
    {
      type: 'apiKey',
      in: 'header',
      name: 'X-API-Key',
    },
    'api-key',
  )
  .addOAuth2(
    {
      type: 'oauth2',
      flows: {
        authorizationCode: {
          authorizationUrl: 'https://auth.example.com/authorize',
          tokenUrl: 'https://auth.example.com/token',
          scopes: {
            'read:users': 'Read user data',
            'write:users': 'Modify user data',
          },
        },
      },
    },
    'oauth2',
  )
  .build();

// In controller
@ApiBearerAuth('JWT-auth')
@ApiSecurity('api-key')
@Get('protected')
protectedRoute() { ... }
```

The second argument to each `add*` method is a name you reference later with `@ApiBearerAuth('JWT-auth')` or `@ApiSecurity('api-key')`.

### Multiple API Documents

Generate separate Swagger documents for different audiences by filtering which modules are included.

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Public API documentation
  const publicConfig = new DocumentBuilder()
    .setTitle('Public API')
    .setVersion('1.0')
    .build();
  const publicDocument = SwaggerModule.createDocument(app, publicConfig, {
    include: [PublicUsersModule, PublicProductsModule],
  });
  SwaggerModule.setup('api/public', app, publicDocument);

  // Internal API documentation
  const internalConfig = new DocumentBuilder()
    .setTitle('Internal API')
    .setVersion('1.0')
    .addBearerAuth()
    .build();
  const internalDocument = SwaggerModule.createDocument(app, internalConfig, {
    include: [AdminModule, InternalModule],
  });
  SwaggerModule.setup('api/internal', app, internalDocument);

  await app.listen(3000);
}
```

The `include` array restricts each document to controllers from the specified modules only.

### Extra Models

Register DTOs that are not directly referenced by any controller but need to appear in the schema.

```typescript
const document = SwaggerModule.createDocument(app, config, {
  extraModels: [ErrorResponseDto, PaginationMetaDto],
});
```

This is commonly needed for shared DTOs used inside `$ref` or `allOf` compositions.

### Generic Response Types

Use `allOf` with `$ref` to document generic wrapper types like paginated responses.

```typescript
export class PaginatedResponseDto<T> {
  @ApiProperty({ isArray: true })
  data: T[];

  @ApiProperty()
  meta: PaginationMetaDto;
}

// In controller
@Get()
@ApiOkResponse({
  schema: {
    allOf: [
      { $ref: getSchemaPath(PaginatedResponseDto) },
      {
        properties: {
          data: {
            type: 'array',
            items: { $ref: getSchemaPath(UserDto) },
          },
        },
      },
    ],
  },
})
findAll(): Promise<PaginatedResponseDto<UserDto>> { ... }
```

Both `PaginatedResponseDto` and `UserDto` must be listed in `extraModels` for `getSchemaPath()` to resolve.

### Custom Swagger Decorator

Combine multiple Swagger decorators into a single reusable decorator with `applyDecorators()`.

```typescript
import { applyDecorators } from '@nestjs/common';
import { ApiOperation, ApiResponse, ApiBearerAuth } from '@nestjs/swagger';

export function ApiAuthEndpoint(summary: string) {
  return applyDecorators(
    ApiBearerAuth(),
    ApiOperation({ summary }),
    ApiResponse({ status: 401, description: 'Unauthorized' }),
    ApiResponse({ status: 403, description: 'Forbidden' }),
  );
}

// Usage
@ApiAuthEndpoint('Get current user profile')
@Get('profile')
getProfile() { ... }
```

This eliminates repetition across dozens of authenticated endpoints.

### File Upload Documentation

Use `@ApiConsumes('multipart/form-data')` and `@ApiBody()` with a binary schema to document file upload endpoints.

```typescript
@Post('upload')
@ApiConsumes('multipart/form-data')
@ApiBody({
  schema: {
    type: 'object',
    properties: {
      file: {
        type: 'string',
        format: 'binary',
      },
      description: {
        type: 'string',
      },
    },
  },
})
@UseInterceptors(FileInterceptor('file'))
uploadFile(@UploadedFile() file: Express.Multer.File) { ... }
```

The `format: 'binary'` tells Swagger UI to render a file picker for that field.

### Hide Endpoints

Use `@ApiExcludeEndpoint()` or `@ApiExcludeController()` to hide internal or debug routes from the documentation.

```typescript
@ApiExcludeEndpoint()
@Get('internal')
internalEndpoint() { ... }

@ApiExcludeController()
@Controller('debug')
export class DebugController { ... }
```

Excluded endpoints still function normally; they are simply omitted from the generated OpenAPI specification.

## Common Pitfalls

1. **Forgetting security decorators**: Document auth even if guards handle it.
2. **Not testing in Swagger UI**: Verify auth flows work.
3. **Stale documentation**: Keep docs in sync with code.

## Best Practices

- Create custom decorators for common patterns
- Separate public and internal API docs
- Test authentication flows in Swagger UI
- Use extraModels for shared DTOs
- Hide internal/debug endpoints from production docs

## Summary

- Configure security schemes (Bearer, API Key, OAuth2) for authenticated endpoint testing
- Create multiple Swagger documents for different audiences (public vs internal)
- Use extraModels to include shared DTOs not directly referenced by controllers
- Build custom decorators that combine multiple Swagger decorators to reduce boilerplate
- Hide internal or debug endpoints from production docs with @ApiExcludeEndpoint()

## Code Examples

**Configuring multiple authentication schemes (Bearer, API Key, OAuth2) with DocumentBuilder**

```typescript
const config = new DocumentBuilder()
  .setTitle('API')
  .addBearerAuth(
    {
      type: 'http',
      scheme: 'bearer',
      bearerFormat: 'JWT',
      description: 'Enter JWT token',
    },
    'JWT-auth', // Name for reference
  )
  .addApiKey(
    {
      type: 'apiKey',
      in: 'header',
      name: 'X-API-Key',
    },
    'api-key',
  )
  .addOAuth2(
    {
      type: 'oauth2',
      flows: {
        authorizationCode: {
          authorizationUrl: 'https://auth.example.com/authorize',
          tokenUrl: 'https://auth.example.com/token',
          scopes: {
            'read:users': 'Read user data',
            'write:users': 'Modify user data',
          },
        },
      },
    },
    'oauth2',
  )
  .build();

// In controller
@ApiBearerAuth('JWT-auth')
@ApiSecurity('api-key')
@Get('protected')
protectedRoute() { ... }
```


## Resources

- [OpenAPI Operations](https://docs.nestjs.com/openapi/operations) — Advanced OpenAPI features

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*