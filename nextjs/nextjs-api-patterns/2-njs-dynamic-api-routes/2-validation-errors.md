---
source_course: "nextjs-api-patterns"
source_lesson: "nextjs-api-patterns-njs-validation-errors"
---

# Input Validation with Zod

## Introduction

Never trust user input. Whether it's a missing field, wrong type, or SQL injection attempt, your API must validate everything. Zod makes this type-safe and declarative.

## Key Concepts

**Zod** is a TypeScript-first schema validation library:

- **Schemas** define what valid data looks like
- **Parsing** validates input and returns typed data
- **Errors** provide detailed field-by-field feedback

## Real World Context

Without validation:
- Invalid data corrupts your database
- Type errors crash your server
- Security vulnerabilities expose your system
- Users get confusing error messages

## Deep Dive

### Basic Schema Definition

```typescript
import { z } from 'zod';

const CreateUserSchema = z.object({
  name: z.string().min(1, 'Name is required').max(100),
  email: z.string().email('Invalid email address'),
  age: z.number().min(18, 'Must be 18 or older').optional(),
  role: z.enum(['user', 'admin']).default('user'),
});

// Infer TypeScript type from schema
type CreateUserInput = z.infer<typeof CreateUserSchema>;
// { name: string; email: string; age?: number; role: 'user' | 'admin' }
```

### Safe Parsing in Route Handlers

```typescript
export async function POST(request: Request) {
  const body = await request.json();
  const result = CreateUserSchema.safeParse(body);

  if (!result.success) {
    return Response.json(
      {
        error: 'Validation failed',
        details: result.error.flatten().fieldErrors,
      },
      { status: 422 }
    );
  }

  // result.data is fully typed and validated!
  const { name, email, age, role } = result.data;
  
  const user = await prisma.user.create({
    data: { name, email, age, role },
  });
  
  return Response.json(user, { status: 201 });
}
```

### Reusable Validation Helper

```typescript
// lib/validation.ts
import { z, ZodSchema } from 'zod';

export async function validateBody<T>(
  request: Request,
  schema: ZodSchema<T>
): Promise<{ data: T } | { error: Response }> {
  try {
    const body = await request.json();
    const result = schema.safeParse(body);
    
    if (!result.success) {
      return {
        error: Response.json(
          {
            error: 'Validation failed',
            details: result.error.flatten().fieldErrors,
          },
          { status: 422 }
        ),
      };
    }
    
    return { data: result.data };
  } catch {
    return {
      error: Response.json(
        { error: 'Invalid JSON body' },
        { status: 400 }
      ),
    };
  }
}

// Usage
export async function POST(request: Request) {
  const validation = await validateBody(request, CreateUserSchema);
  
  if ('error' in validation) {
    return validation.error;
  }
  
  const { name, email } = validation.data;
  // Continue with validated data...
}
```

### Common Schema Patterns

```typescript
// String validations
z.string().min(1).max(255)           // Length limits
z.string().email()                    // Email format
z.string().url()                      // URL format
z.string().uuid()                     // UUID format
z.string().regex(/^[a-z]+$/)          // Custom regex

// Number validations
z.number().int()                      // Integer only
z.number().positive()                 // > 0
z.number().min(0).max(100)            // Range

// Arrays and objects
z.array(z.string())                   // Array of strings
z.object({ ... }).strict()            // No extra fields allowed
z.record(z.number())                  // { [key: string]: number }

// Transformations
z.string().transform(s => s.toLowerCase())
z.coerce.number()                     // "123" -> 123
```

### Update vs Create Schemas

```typescript
const CreatePostSchema = z.object({
  title: z.string().min(1),
  content: z.string().min(1),
  published: z.boolean().default(false),
});

// Make all fields optional for PATCH
const UpdatePostSchema = CreatePostSchema.partial();

// Or pick specific fields
const PublishPostSchema = CreatePostSchema.pick({ published: true });
```

## Common Pitfalls

1. **Using `.parse()` instead of `.safeParse()`**: `.parse()` throws on error. Use `.safeParse()` in APIs to handle errors gracefully.

2. **Not validating params**: URL params like `[id]` should also be validated: `z.string().uuid().parse(params.id)`.

3. **Over-trusting schemas**: Schemas validate structure, not business logic. Check for duplicates, permissions, etc. separately.

## Best Practices

- **Create schema files**: Keep schemas in `schemas/` or alongside your routes
- **Use `.safeParse()`**: Never let validation throw uncaught exceptions
- **Return field errors**: Clients need to know which field failed
- **Use `.coerce` for query params**: They're always strings initially

## Summary

Zod provides type-safe validation for your API inputs. Define schemas to describe valid data, use `.safeParse()` to validate without throwing, and return detailed field errors. Create reusable validation helpers for consistent error handling across your API.

## Resources

- [Zod Documentation](https://zod.dev/) — Official Zod documentation

---

> 📘 *This lesson is part of the [Next.js API Routes & Patterns](https://stanza.dev/courses/nextjs-api-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*