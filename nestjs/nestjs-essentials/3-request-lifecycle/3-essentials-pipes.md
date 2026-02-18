---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-pipes"
---

# Pipes

## Introduction

Data coming into your API needs to be validated and transformed before your business logic touches it. Pipes are the gatekeepers—they validate input data, transform types, and reject invalid requests before they reach your route handlers.

## Key Concepts

- **Pipe**: A class that transforms or validates input data
- **ValidationPipe**: Built-in pipe for DTO validation using class-validator
- **Transformation**: Converting data types (string to number, string to Date)
- **Validation**: Ensuring data meets expected constraints

## Real World Context

Consider a `GET /users/:id` endpoint. The `:id` parameter comes as a string from the URL. Your service expects a number. Pipes automatically convert "123" to 123, or throw a 400 Bad Request if the value is "abc".

## Deep Dive

### Built-in Pipes

| Pipe | Purpose |
|------|---------|
| `ValidationPipe` | Validates DTOs using class-validator decorators |
| `ParseIntPipe` | Converts string to integer |
| `ParseBoolPipe` | Converts string to boolean |
| `ParseUUIDPipe` | Validates UUID format |
| `ParseArrayPipe` | Parses and validates arrays |
| `DefaultValuePipe` | Provides default when value is undefined |

### Using ParseIntPipe

```typescript
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {
  // id is guaranteed to be a number
  return this.usersService.findOne(id);
}
```

If called with `/users/abc`, returns:
```json
{
  "statusCode": 400,
  "message": "Validation failed (numeric string is expected)",
  "error": "Bad Request"
}
```

### ValidationPipe with DTOs

```typescript
// Enable globally in main.ts
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,
  forbidNonWhitelisted: true,
  transform: true,
}));

// Use with @Body()
@Post()
create(@Body() createUserDto: CreateUserDto) {
  return this.usersService.create(createUserDto);
}
```

### Custom Pipes

```typescript
import { PipeTransform, Injectable, BadRequestException } from '@nestjs/common';

@Injectable()
export class ParseDatePipe implements PipeTransform<string, Date> {
  transform(value: string): Date {
    const date = new Date(value);
    if (isNaN(date.getTime())) {
      throw new BadRequestException('Invalid date format');
    }
    return date;
  }
}
```

### Combining Multiple Pipes

```typescript
@Get(':id')
findOne(
  @Param('id', ParseIntPipe) id: number,
  @Query('date', new DefaultValuePipe(new Date().toISOString()), ParseDatePipe) date: Date,
) {
  return this.service.findOne(id, date);
}
```

## Common Pitfalls

1. **Forgetting `transform: true`**: Without it, `ParseIntPipe` works but types remain strings in complex DTOs.
2. **Validation after business logic**: Validate at the controller boundary, not deep in services.
3. **Over-customizing pipes**: Built-in pipes cover most cases. Custom pipes should be rare.

## Best Practices

- Enable `ValidationPipe` globally with `whitelist: true`
- Use `ParseIntPipe` for all numeric route parameters
- Create custom pipes only for domain-specific transformations
- Combine pipes for complex parameter handling

## Summary

Pipes validate and transform input data before it reaches your handlers. Built-in pipes handle common cases like `ParseIntPipe` and `ValidationPipe`. Custom pipes handle domain-specific transformations. Enable them globally for consistent validation across your API.

## Resources

- [Pipes Documentation](https://docs.nestjs.com/pipes) — Official guide to pipes and validation

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*