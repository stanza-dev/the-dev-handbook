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

## Real World Context

Invalid input is the #1 source of bugs and security vulnerabilities. A user submitting an email field with a script tag, a negative quantity on an order, or a missing required field can crash your app or worse. Validation at the API boundary catches these issues before they reach your business logic.

## Deep Dive

### Global Validation Setup

Apply `ValidationPipe` globally in `main.ts` so every incoming request is validated automatically.

```typescript
// main.ts
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,
  forbidNonWhitelisted: true,
  transform: true,
  transformOptions: { enableImplicitConversion: true },
}));
```

The `whitelist` option strips unknown properties, and `forbidNonWhitelisted` rejects requests containing them.

### DTO with Validation

Decorate DTO properties with `class-validator` decorators to declare constraints declaratively.

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

Stack multiple decorators on a single property to combine constraints, e.g. `@IsInt()` and `@Min(0)` together.

### Nested Validation

Use `@ValidateNested()` with `@Type()` from `class-transformer` to validate objects inside arrays or nested fields.

```typescript
import { ValidateNested, Type } from 'class-transformer';

export class CreateOrderDto {
  @ValidateNested({ each: true })
  @Type(() => OrderItemDto)
  items: OrderItemDto[];
}
```

Without `@Type()`, the nested objects remain plain objects and their decorators are never evaluated.

### Custom Validation

Create custom validator classes by implementing `ValidatorConstraintInterface` for business-specific rules like uniqueness checks.

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

Setting `async: true` in `@ValidatorConstraint` allows the validator to return a Promise, enabling database lookups during validation.

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

## Code Examples

**Global ValidationPipe setup with whitelist, transform, and strict mode options**

```typescript
// main.ts
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,
  forbidNonWhitelisted: true,
  transform: true,
  transformOptions: { enableImplicitConversion: true },
}));
```


## Resources

- [Validation](https://docs.nestjs.com/techniques/validation) — Official validation guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*