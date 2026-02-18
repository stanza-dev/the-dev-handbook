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

### Custom Validators

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

### Global Validation Configuration

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

### Array and Nested Validation

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

Advanced validation uses whitelisting, custom validators, and proper transformation. Enable `whitelist` and `forbidNonWhitelisted` globally. Create custom validators for domain-specific rules. Validate nested objects and arrays with proper decorators.

## Resources

- [Validation](https://docs.nestjs.com/techniques/validation) — Official validation guide
- [class-validator](https://github.com/typestack/class-validator) — Validation decorator reference

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*