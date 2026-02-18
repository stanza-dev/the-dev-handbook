---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-serialization"
---

# Serialization

## Introduction

Not all data should be sent to clients. Passwords, internal IDs, and sensitive fields need to be excluded from responses. Serialization transforms your data before it reaches the client.

## Key Concepts

- **ClassSerializerInterceptor**: Applies class-transformer to responses
- **@Exclude()**: Remove property from serialization
- **@Expose()**: Explicitly include property
- **@Transform()**: Custom transformation logic

## Deep Dive

### Setup

```typescript
// main.ts
app.useGlobalInterceptors(new ClassSerializerInterceptor(app.get(Reflector)));
```

### Entity with Serialization

```typescript
import { Exclude, Expose, Transform } from 'class-transformer';

export class UserEntity {
  id: string;
  email: string;

  @Exclude()
  password: string;

  @Exclude()
  internalNotes: string;

  @Expose({ name: 'fullName' })
  getFullName(): string {
    return \`\${this.firstName} \${this.lastName}\`;
  }

  @Transform(({ value }) => value.toISOString())
  createdAt: Date;

  constructor(partial: Partial<UserEntity>) {
    Object.assign(this, partial);
  }
}
```

### Using in Controller

```typescript
@Get(':id')
findOne(@Param('id') id: string): UserEntity {
  const user = this.usersService.findOne(id);
  return new UserEntity(user);
}
```

### Serialization Groups

```typescript
export class UserEntity {
  id: string;
  
  @Expose({ groups: ['admin'] })
  internalId: string;
  
  @Expose({ groups: ['admin', 'owner'] })
  email: string;
}

// In controller
@SerializeOptions({ groups: ['admin'] })
@Get('admin/users')
findAllForAdmin() { ... }
```

### Response Mapping

```typescript
// Create separate response DTOs
export class UserResponseDto {
  id: string;
  email: string;
  name: string;

  constructor(user: User) {
    this.id = user.id;
    this.email = user.email;
    this.name = user.name;
  }
}
```

## Common Pitfalls

1. **Not wrapping in entity/dto**: Raw objects aren't transformed. Use `new Entity(data)`.
2. **Circular references**: Nested entities can cause infinite loops. Use `@Transform()` or separate DTOs.
3. **Performance**: Complex transformations on large datasets can be slow.

## Best Practices

- Create separate response DTOs for API contracts
- Use groups for role-based response variations
- Keep transformations simple
- Document which fields are excluded

## Summary

Serialization transforms response data using class-transformer decorators. @Exclude() hides sensitive fields, @Expose() adds computed properties, and @Transform() applies custom logic. Use separate response DTOs for clean API contracts.

## Resources

- [Serialization](https://docs.nestjs.com/techniques/serialization) — Official serialization guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*