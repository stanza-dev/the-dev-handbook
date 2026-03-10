---
source_course: "nestjs-security"
source_lesson: "nestjs-security-input-validation-deep"
---

# Advanced Input Validation

## Introduction

Input validation is your first line of defense against attacks. While basic validation checks types and lengths, advanced validation prevents SQL injection, XSS, command injection, and business logic attacks. Every byte from the client is potentially malicious.

## Key Concepts

- **Whitelisting**: Accept only known-good input patterns
- **Blacklisting**: Block known-bad patterns (less secure)
- **Sanitization**: Clean input to remove dangerous content
- **Validation vs Sanitization**: Validation rejects bad input; sanitization cleans it

## Real World Context

Validation failures have caused major breaches:
- SQL injection via unvalidated search fields
- XSS through user profile fields
- Command injection through file upload names
- Path traversal via URL parameters

## Deep Dive

### Comprehensive DTO Validation

Combine multiple decorators on each field to enforce type, length, and format constraints. Custom error messages make debugging easier for API consumers.

```typescript
import {
  IsString,
  IsEmail,
  MinLength,
  MaxLength,
  Matches,
  IsUUID,
  IsOptional,
  ValidateIf,
} from 'class-validator';

export class CreateUserDto {
  @IsEmail({}, { message: 'Invalid email format' })
  @MaxLength(255)
  email: string;

  @IsString()
  @MinLength(8, { message: 'Password must be at least 8 characters' })
  @MaxLength(128)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, {
    message: 'Password must contain uppercase, lowercase, and number',
  })
  password: string;

  @IsString()
  @MinLength(1)
  @MaxLength(100)
  @Matches(/^[a-zA-Z\s'-]+$/, {
    message: 'Name contains invalid characters',
  })
  name: string;

  @IsOptional()
  @IsUUID('4')
  referralCode?: string;
}
```

The `@Matches()` regex on the name field whitelists only alphabetic characters, spaces, hyphens, and apostrophes — blocking injection attempts at the DTO level.

### Custom Validators

Create reusable custom validators for domain-specific rules like SQL injection detection. This adds defense in depth alongside parameterized queries.

```typescript
import {
  ValidatorConstraint,
  ValidatorConstraintInterface,
  ValidationArguments,
  Validate,
} from 'class-validator';

@ValidatorConstraint({ name: 'noSqlInjection', async: false })
export class NoSqlInjectionConstraint implements ValidatorConstraintInterface {
  private readonly dangerousPatterns = [
    /('|")?\s*(or|and)\s+\d+\s*=\s*\d+/i,
    /union\s+select/i,
    /;\s*drop\s+table/i,
    /--\s*$/,
  ];

  validate(text: string): boolean {
    if (typeof text !== 'string') return true;
    return !this.dangerousPatterns.some(pattern => pattern.test(text));
  }

  defaultMessage(): string {
    return 'Input contains potentially dangerous content';
  }
}

export class SearchDto {
  @IsString()
  @Validate(NoSqlInjectionConstraint)
  query: string;
}
```

This validator catches common injection patterns like `OR 1=1`, `UNION SELECT`, and comment-based bypasses. Apply it to any free-text search fields.

### Global Validation Configuration

Configure the global `ValidationPipe` with strict settings. The `whitelist` option strips unexpected fields, and `forbidNonWhitelisted` rejects them outright.

```typescript
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,              // Strip unknown properties
    forbidNonWhitelisted: true,   // Throw on unknown properties
    transform: true,              // Auto-transform types
    transformOptions: {
      enableImplicitConversion: true,
    },
    stopAtFirstError: false,      // Return all errors
    exceptionFactory: (errors) => {
      const messages = errors.flatMap((error) =>
        Object.values(error.constraints || {}),
      );
      return new BadRequestException({
        statusCode: 400,
        message: 'Validation failed',
        errors: messages,
      });
    },
  }),
);
```

The custom `exceptionFactory` returns all validation errors at once, so API consumers can fix multiple issues in a single request round-trip.

### Array and Nested Validation

Use `@ValidateNested()` with `@Type()` to validate objects inside arrays. Always set array size limits to prevent DoS via massive payloads.

```typescript
import { Type } from 'class-transformer';
import { ValidateNested, ArrayMinSize, ArrayMaxSize } from 'class-validator';

export class OrderItemDto {
  @IsUUID()
  productId: string;

  @IsInt()
  @Min(1)
  @Max(100)
  quantity: number;
}

export class CreateOrderDto {
  @ValidateNested({ each: true })
  @Type(() => OrderItemDto)
  @ArrayMinSize(1)
  @ArrayMaxSize(50)
  items: OrderItemDto[];
}
```

The `@Type(() => OrderItemDto)` is critical — without it, `class-transformer` cannot instantiate the nested objects, and validation decorators on `OrderItemDto` will be silently ignored.

## Common Pitfalls

1. **Client-side only validation**: Always validate server-side. Client validation is UX, not security.
2. **Trusting content-type**: Attackers can send any content-type. Validate the actual content.
3. **Allowing too much**: Whitelist what you need rather than blacklisting what you don't.

## Best Practices

- Enable whitelist mode to strip unexpected fields
- Use strict regex patterns for formatted data (emails, phones)
- Validate array lengths to prevent DoS
- Return all validation errors, not just the first
- Log validation failures for security monitoring

## Summary

- Enable `whitelist` and `forbidNonWhitelisted` globally to strip or reject unexpected properties
- Create custom validators for domain-specific rules like SQL injection detection
- Validate nested objects with `@ValidateNested()` and arrays with `@ArrayMinSize()`/`@ArrayMaxSize()`
- Always validate server-side—client-side validation is for UX, not security

## Code Examples

**Advanced ValidationPipe configuration with custom error formatting**

```typescript
import {
  IsString,
  IsEmail,
  MinLength,
  MaxLength,
  Matches,
  IsUUID,
  IsOptional,
  ValidateIf,
} from 'class-validator';

export class CreateUserDto {
  @IsEmail({}, { message: 'Invalid email format' })
  @MaxLength(255)
  email: string;

  @IsString()
  @MinLength(8, { message: 'Password must be at least 8 characters' })
  @MaxLength(128)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, {
    message: 'Password must contain uppercase, lowercase, and number',
  })
  password: string;

  @IsString()
  @MinLength(1)
  @MaxLength(100)
  @Matches(/^[a-zA-Z\s'-]+$/, {
    message: 'Name contains invalid characters',
  })
  name: string;

  @IsOptional()
  @IsUUID('4')
  referralCode?: string;
}
```


## Resources

- [Validation](https://docs.nestjs.com/techniques/validation) — Official validation guide
- [class-validator](https://github.com/typestack/class-validator) — Validation decorator reference

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*