---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-api-responses"
---

# Standardizing API Responses

## Introduction

Consistent API responses improve developer experience. Whether returning data, errors, or metadata, a standard format makes APIs predictable and easier to consume.

## Key Concepts

- **Response Envelope**: Wrapper structure for all responses
- **Error Format**: Consistent error response structure
- **Pagination**: Standard format for paginated data
- **Response Transformation**: Interceptors for response formatting

## Real World Context

Standard responses enable:
- Consistent client-side handling
- Easier SDK generation
- Better error handling
- Predictable pagination

## Deep Dive

### Standard Response Format

```typescript
export class ApiResponse<T> {
  @ApiProperty()
  success: boolean;

  @ApiProperty()
  data: T;

  @ApiPropertyOptional()
  message?: string;

  @ApiPropertyOptional()
  meta?: Record<string, any>;
}
```

### Response Transformation Interceptor

```typescript
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, ApiResponse<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<ApiResponse<T>> {
    return next.handle().pipe(
      map(data => ({
        success: true,
        data,
        timestamp: new Date().toISOString(),
      })),
    );
  }
}

// Apply globally
app.useGlobalInterceptors(new TransformInterceptor());
```

### Standard Error Response

```typescript
export class ErrorResponse {
  @ApiProperty()
  success: false;

  @ApiProperty()
  statusCode: number;

  @ApiProperty()
  message: string;

  @ApiPropertyOptional()
  errorCode?: string;

  @ApiPropertyOptional({ type: [String] })
  errors?: string[];

  @ApiProperty()
  timestamp: string;

  @ApiProperty()
  path: string;
}

@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();

    const status = exception instanceof HttpException
      ? exception.getStatus()
      : HttpStatus.INTERNAL_SERVER_ERROR;

    const errorResponse: ErrorResponse = {
      success: false,
      statusCode: status,
      message: this.getMessage(exception),
      errorCode: this.getErrorCode(exception),
      errors: this.getValidationErrors(exception),
      timestamp: new Date().toISOString(),
      path: request.url,
    };

    response.status(status).json(errorResponse);
  }
}
```

### Pagination Response

```typescript
export class PaginationMeta {
  @ApiProperty()
  page: number;

  @ApiProperty()
  limit: number;

  @ApiProperty()
  total: number;

  @ApiProperty()
  totalPages: number;

  @ApiProperty()
  hasNextPage: boolean;

  @ApiProperty()
  hasPreviousPage: boolean;
}

export class PaginatedResponse<T> {
  @ApiProperty()
  success: true;

  data: T[];

  @ApiProperty({ type: PaginationMeta })
  meta: PaginationMeta;
}

// Helper function
export function createPaginatedResponse<T>(
  data: T[],
  total: number,
  page: number,
  limit: number,
): PaginatedResponse<T> {
  const totalPages = Math.ceil(total / limit);
  return {
    success: true,
    data,
    meta: {
      page,
      limit,
      total,
      totalPages,
      hasNextPage: page < totalPages,
      hasPreviousPage: page > 1,
    },
  };
}
```

### Usage in Controller

```typescript
@Get()
@ApiOkResponse({ type: PaginatedResponse })
async findAll(
  @Query() query: PaginationQueryDto,
): Promise<PaginatedResponse<UserDto>> {
  const [users, total] = await this.usersService.findAll(query);
  return createPaginatedResponse(
    users.map(u => new UserDto(u)),
    total,
    query.page,
    query.limit,
  );
}
```

## Common Pitfalls

1. **Inconsistent formats**: All endpoints should use the same structure.
2. **Missing error details**: Include enough info for debugging.
3. **Not documenting**: Swagger should show the actual response format.

## Best Practices

- Use a consistent envelope for all responses
- Include success boolean for easy checking
- Standardize pagination metadata
- Document response format in Swagger
- Include timestamps and request paths in errors

## Summary

Standard API responses use consistent structure for success, error, and paginated data. Use interceptors for automatic wrapping, exception filters for error formatting, and helper functions for pagination. Document formats in Swagger.

## Resources

- [Interceptors](https://docs.nestjs.com/interceptors) — Response transformation

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*