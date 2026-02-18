---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-validation"
---

# Validation with Pipes

## Introduction

User input is unpredictable. Validation ensures data meets your expectations before it reaches business logic. NestJS's ValidationPipe, combined with class-validator, provides declarative, powerful validation.

## Key Concepts

- **ValidationPipe**: Built-in pipe that validates against DTOs
- **class-validator**: Decorator-based validation library
- **Whitelist**: Strip unknown properties from requests
- **Transform**: Automatically convert types

## Deep Dive

### Global Validation Setup

```typescript
// main.ts
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,
  forbidNonWhitelisted: true,
  transform: true,
  transformOptions: { enableImplicitConversion: true },
}));
```

### DTO with Validation

```typescript
import { IsEmail, IsString, MinLength, IsOptional, IsInt, Min } from 'class-validator';

export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;

  @IsString()
  @IsOptional()
  name?: string;

  @IsInt()
  @Min(0)
  @IsOptional()
  age?: number;
}
```

### Nested Validation

```typescript
import { ValidateNested, Type } from 'class-transformer';

export class CreateOrderDto {
  @ValidateNested({ each: true })
  @Type(() => OrderItemDto)
  items: OrderItemDto[];
}
```

### Custom Validation

```typescript
import { ValidatorConstraint, ValidatorConstraintInterface, Validate } from 'class-validator';

@ValidatorConstraint({ async: true })
class IsUniqueEmail implements ValidatorConstraintInterface {
  validate(email: string) {
    return UserRepository.findOne({ email }).then(user => !user);
  }
}

export class CreateUserDto {
  @Validate(IsUniqueEmail, { message: 'Email already exists' })
  email: string;
}
```

## Common Pitfalls

1. **Missing transform: true**: Types stay as strings without transformation.
2. **Forgetting @Type()**: Nested objects need explicit type information.
3. **Not enabling globally**: Validation only works where ValidationPipe is applied.

## Best Practices

- Enable ValidationPipe globally with whitelist
- Create separate DTOs for create/update operations
- Use class-transformer for complex transformations
- Provide clear validation error messages

## Summary

ValidationPipe validates DTOs using class-validator decorators. Enable globally with whitelist and transform options. Use @ValidateNested for nested objects and custom validators for business rules.

## Resources

- [Validation](https://docs.nestjs.com/techniques/validation) — Official validation guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*