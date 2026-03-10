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

Define a generic envelope class that wraps every successful response with a `success` flag, `data`, and optional metadata.

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

Clients can always check `response.success` before accessing data, simplifying error handling.

### Response Transformation Interceptor

A global interceptor wraps every controller return value into the standard envelope automatically.

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

The `map()` operator transforms the response after the handler completes but before it is sent to the client.

### Standard Error Response

Pair the success envelope with a consistent error format using a global exception filter.

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

The `@Catch()` decorator with no arguments catches every exception type, including non-HTTP errors.

### Pagination Response

Define reusable pagination metadata and a helper function to build paginated responses consistently.

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

The `hasNextPage` and `hasPreviousPage` booleans let clients implement pagination controls without calculating page counts.

### Usage in Controller

Call the helper function in your controller to return a fully formatted paginated response.

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

The service returns a tuple of `[data, total]`, keeping pagination logic out of the data access layer.

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

- Use a consistent response envelope with success, data, and optional meta fields
- Apply a TransformInterceptor globally to wrap all responses automatically
- Standardize error responses with GlobalExceptionFilter for consistent error formatting
- Create helper functions for paginated responses with metadata (page, total, hasNext)
- Document all response formats in Swagger for clear API contracts

## Code Examples

**Defining a generic ApiResponse class with success, data, message, and meta fields**

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


## Resources

- [Interceptors](https://docs.nestjs.com/interceptors) — Response transformation

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*