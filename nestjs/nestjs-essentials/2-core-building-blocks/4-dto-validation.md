---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-dto-validation"
---

# DTOs and Request Validation

## Introduction

Never trust user input. Every API endpoint that accepts data needs validation to prevent crashes, security vulnerabilities, and data corruption. NestJS provides a powerful validation pipeline using DTOs (Data Transfer Objects) and class-validator.

## Key Concepts

- **DTO (Data Transfer Object)**: A class that defines the shape of data for a specific operation
- **ValidationPipe**: Built-in pipe that validates incoming data against DTO rules
- **class-validator**: Decorator-based validation library
- **class-transformer**: Transforms plain objects to class instances

## Real World Context

Imagine a user registration endpoint. Without validation:
- Email could be `"not-an-email"`
- Password could be empty
- Age could be `-5`

With proper validation, these requests are rejected **before** reaching your business logic, with clear error messages for the client.

## Deep Dive

### Setting Up Validation

```bash
npm install class-validator class-transformer
```

Enable globally in `main.ts`:

```typescript
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe({
    whitelist: true,       // Strip unknown properties
    forbidNonWhitelisted: true, // Throw on unknown properties
    transform: true,       // Auto-transform to DTO types
  }));
  await app.listen(3000);
}
```

### Creating a DTO

```typescript
import { IsString, IsEmail, IsInt, Min, Max, MinLength } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @MinLength(2)
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;

  @IsInt()
  @Min(0)
  @Max(120)
  age: number;
}
```

### Using DTOs in Controllers

```typescript
@Controller('users')
export class UsersController {
  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    // createUserDto is validated and typed!
    return this.usersService.create(createUserDto);
  }
}
```

### Common Validators

| Decorator | Validates |
|-----------|----------|
| `@IsString()` | Value is a string |
| `@IsInt()` | Value is an integer |
| `@IsEmail()` | Valid email format |
| `@IsOptional()` | Field can be undefined |
| `@MinLength(n)` | String has minimum length |
| `@Min(n)` / `@Max(n)` | Number range |
| `@IsArray()` | Value is an array |
| `@ValidateNested()` | Validate nested objects |

### Validation Error Response

When validation fails, NestJS returns:

```json
{
  "statusCode": 400,
  "message": [
    "email must be an email",
    "password must be longer than or equal to 8 characters"
  ],
  "error": "Bad Request"
}
```

## Common Pitfalls

1. **Forgetting `transform: true`**: Without it, `@Param('id')` stays a string even with `id: number` type annotation.
2. **Not using `whitelist`**: Without it, clients can inject extra properties that pass through to your service.
3. **Validating in services**: Validation belongs at the controller boundary, not buried in business logic.

## Best Practices

- **One DTO per operation**: `CreateUserDto`, `UpdateUserDto`, `UserResponseDto`
- **Use `PartialType()` for updates**: `export class UpdateUserDto extends PartialType(CreateUserDto) {}`
- **Separate input and output DTOs**: Don't expose internal fields (passwords, internal IDs)
- **Validate everything**: Query params, path params, headers—not just body

## Summary

DTOs define expected data shapes, and class-validator decorators enforce rules. The ValidationPipe validates automatically when enabled globally. This ensures all incoming data is validated before reaching your business logic, improving security and reliability.

## Resources

- [Validation Documentation](https://docs.nestjs.com/techniques/validation) — Official guide to request validation with ValidationPipe
- [class-validator Decorators](https://github.com/typestack/class-validator#validation-decorators) — Complete list of validation decorators

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*