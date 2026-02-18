---
source_course: "nestjs-security"
source_lesson: "nestjs-security-request-validation"
---

# Request-Level Security

## Introduction

Beyond body validation, secure applications validate headers, query parameters, content types, and request sizes. Attackers exploit any input vector—not just POST bodies.

## Key Concepts

- **Content-Type Validation**: Ensure requests have expected content types
- **Request Size Limits**: Prevent DoS via large payloads
- **Header Validation**: Validate custom headers and tokens
- **Query Parameter Validation**: Type-safe query string handling

## Real World Context

Request-level attacks include:
- Large payload DoS attacks
- Content-type confusion attacks
- Header injection
- Parameter pollution

## Deep Dive

### Body Size Limits

```typescript
// Express
import * as bodyParser from 'body-parser';

app.use(bodyParser.json({ limit: '1mb' }));
app.use(bodyParser.urlencoded({ limit: '1mb', extended: true }));

// Fastify
new FastifyAdapter({
  bodyLimit: 1048576, // 1MB in bytes
});
```

### Query Parameter Validation

```typescript
import { IsOptional, IsInt, Min, Max, IsEnum, IsDateString } from 'class-validator';
import { Type } from 'class-transformer';

export enum SortOrder {
  ASC = 'asc',
  DESC = 'desc',
}

export class PaginationQueryDto {
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  page?: number = 1;

  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  limit?: number = 10;

  @IsOptional()
  @IsString()
  @MaxLength(50)
  sortBy?: string;

  @IsOptional()
  @IsEnum(SortOrder)
  sortOrder?: SortOrder = SortOrder.DESC;
}

@Get()
findAll(@Query() query: PaginationQueryDto) {
  // query is validated and transformed
}
```

### Header Validation

```typescript
import { createParamDecorator, ExecutionContext, BadRequestException } from '@nestjs/common';

export const ApiVersion = createParamDecorator(
  (data: unknown, ctx: ExecutionContext): string => {
    const request = ctx.switchToHttp().getRequest();
    const version = request.headers['x-api-version'];
    
    if (!version || !/^v[1-9]$/.test(version)) {
      throw new BadRequestException('Invalid or missing X-API-Version header');
    }
    
    return version;
  },
);

@Get()
findAll(@ApiVersion() version: string) {
  // version is validated
}
```

### Content-Type Guard

```typescript
@Injectable()
export class JsonContentTypeGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const method = request.method;
    
    // Only check for methods with bodies
    if (['POST', 'PUT', 'PATCH'].includes(method)) {
      const contentType = request.headers['content-type'];
      if (!contentType?.includes('application/json')) {
        throw new UnsupportedMediaTypeException('Content-Type must be application/json');
      }
    }
    
    return true;
  }
}
```

### File Upload Validation

```typescript
import { ParseFilePipe, MaxFileSizeValidator, FileTypeValidator } from '@nestjs/common';

@Post('upload')
@UseInterceptors(FileInterceptor('file'))
uploadFile(
  @UploadedFile(
    new ParseFilePipe({
      validators: [
        new MaxFileSizeValidator({ maxSize: 5 * 1024 * 1024 }), // 5MB
        new FileTypeValidator({ fileType: /^image\/(jpeg|png|gif)$/ }),
      ],
      fileIsRequired: true,
    }),
  )
  file: Express.Multer.File,
) {
  return { filename: file.originalname, size: file.size };
}
```

### Rate Limiting per Endpoint

```typescript
import { Throttle, SkipThrottle } from '@nestjs/throttler';

@Controller('auth')
export class AuthController {
  @Throttle({ default: { limit: 5, ttl: 60000 } }) // 5 per minute
  @Post('login')
  login(@Body() dto: LoginDto) { ... }

  @Throttle({ default: { limit: 3, ttl: 3600000 } }) // 3 per hour
  @Post('password-reset')
  resetPassword(@Body() dto: PasswordResetDto) { ... }

  @SkipThrottle()
  @Get('verify-email/:token')
  verifyEmail(@Param('token') token: string) { ... }
}
```

## Common Pitfalls

1. **No body size limits**: Attackers can send gigabyte requests without limits.
2. **Trusting query params**: Query strings are user input too—validate them.
3. **Ignoring content-type**: Attackers may send XML when you expect JSON.

## Best Practices

- Set body size limits appropriate for your endpoints
- Validate query parameters with DTOs
- Use ParseFilePipe for file uploads
- Apply stricter rate limits to sensitive endpoints
- Validate content-type for body-accepting methods

## Summary

Request-level security validates all input vectors: body size, content type, headers, and query parameters. Use DTOs for query validation, guards for header validation, and ParseFilePipe for uploads. Set appropriate size limits and rate limits.

## Resources

- [Validation](https://docs.nestjs.com/techniques/validation) — Request validation patterns

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*