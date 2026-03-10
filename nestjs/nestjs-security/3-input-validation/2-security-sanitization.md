---
source_course: "nestjs-security"
source_lesson: "nestjs-security-sanitization"
---

# Input Sanitization

## Introduction

Sanitization transforms input to remove or escape dangerous content. While validation rejects bad input, sanitization cleans it. This is essential for user-generated content that will be stored and displayed.

## Key Concepts

- **HTML Sanitization**: Remove/escape HTML tags to prevent XSS
- **SQL Escaping**: Handled by ORM/query builders (parameterized queries)
- **Path Sanitization**: Prevent directory traversal attacks
- **Transformation**: class-transformer decorators for cleaning input

## Real World Context

Sanitization scenarios:
- User bio fields displayed on profile pages
- Comments and reviews shown to other users
- File uploads with user-provided names
- Search queries displayed in results

## Deep Dive

### HTML/XSS Sanitization

Install `sanitize-html` to strip dangerous HTML tags while preserving safe formatting.

```bash
npm install sanitize-html
npm install -D @types/sanitize-html
```

Wrap it in a reusable `class-transformer` decorator so it can be applied declaratively to any DTO field.

```typescript
import * as sanitizeHtml from 'sanitize-html';
import { Transform } from 'class-transformer';

export function SanitizeHtml() {
  return Transform(({ value }) => {
    if (typeof value !== 'string') return value;
    return sanitizeHtml(value, {
      allowedTags: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
      allowedAttributes: {
        a: ['href', 'title'],
      },
      allowedSchemes: ['http', 'https'],
    });
  });
}

export class CreatePostDto {
  @IsString()
  @MaxLength(100)
  title: string;

  @IsString()
  @MaxLength(10000)
  @SanitizeHtml()
  content: string;
}
```

The `allowedSchemes` option restricts links to `http` and `https`, blocking `javascript:` URIs that could execute code.

### Strip HTML Completely

For fields that should never contain HTML (like comments or usernames), strip all tags by setting `allowedTags` to an empty array.

```typescript
export function StripHtml() {
  return Transform(({ value }) => {
    if (typeof value !== 'string') return value;
    return sanitizeHtml(value, {
      allowedTags: [],
      allowedAttributes: {},
    });
  });
}

export class CommentDto {
  @IsString()
  @StripHtml()
  text: string;
}
```

This is the safest option for plain-text fields — any `<script>`, `<img onerror>`, or other injection attempts are completely removed.

### Trim and Normalize

Create transformer decorators for common cleaning operations like trimming whitespace and normalizing email case.

```typescript
import { Transform } from 'class-transformer';

export function Trim() {
  return Transform(({ value }) =>
    typeof value === 'string' ? value.trim() : value,
  );
}

export function NormalizeEmail() {
  return Transform(({ value }) =>
    typeof value === 'string' ? value.toLowerCase().trim() : value,
  );
}

export class LoginDto {
  @IsEmail()
  @NormalizeEmail()
  email: string;

  @IsString()
  @Trim()
  password: string;
}
```

Normalizing emails to lowercase prevents users from accidentally creating duplicate accounts with different casing.

### Path Sanitization

Sanitize user-provided filenames to prevent directory traversal attacks (e.g., `../../etc/passwd`). Always validate the resolved path stays within your upload directory.

```typescript
import * as path from 'path';

export function sanitizeFilename(filename: string): string {
  // Remove path components
  const basename = path.basename(filename);
  
  // Remove dangerous characters
  return basename
    .replace(/[^a-zA-Z0-9._-]/g, '_')
    .replace(/\.{2,}/g, '.')
    .substring(0, 255);
}

@Injectable()
export class FileService {
  async saveFile(file: Express.Multer.File, userFilename: string) {
    const safeName = sanitizeFilename(userFilename);
    const uniqueName = `${Date.now()}-${safeName}`;
    
    // Always use path.join with validated base directory
    const safePath = path.join(this.uploadDir, uniqueName);
    
    // Verify path is within allowed directory
    if (!safePath.startsWith(this.uploadDir)) {
      throw new BadRequestException('Invalid file path');
    }
    
    await fs.writeFile(safePath, file.buffer);
    return uniqueName;
  }
}
```

The `safePath.startsWith(this.uploadDir)` check is the critical line — it prevents path traversal even if the sanitization function misses an edge case.

### SQL Injection Prevention

Never concatenate user input into SQL strings. Use parameterized queries or ORM methods that handle escaping automatically.

```typescript
// WRONG - Vulnerable to SQL injection
const query = `SELECT * FROM users WHERE name = '${name}'`;

// CORRECT - Parameterized query (TypeORM)
const users = await this.userRepository
  .createQueryBuilder('user')
  .where('user.name = :name', { name })
  .getMany();

// CORRECT - Repository method
const user = await this.userRepository.findOne({ where: { name } });
```

Both correct approaches bind the `name` value as a parameter, so the database never interprets it as SQL syntax — even if it contains malicious content.

## Common Pitfalls

1. **Sanitizing after storage**: Sanitize before storing, not when displaying. Stored XSS is dangerous.
2. **Over-sanitization**: Don't break legitimate content. Allow safe HTML when needed.
3. **Forgetting URL params**: Query strings and path params need validation too.

## Best Practices

- Sanitize user-generated content before storage
- Use parameterized queries (never string concatenation)
- Validate file paths are within allowed directories
- Create reusable transformer decorators
- Configure sanitize-html based on your content needs

## Summary

- Sanitization cleans input to remove dangerous content, complementing validation which rejects it
- Use `sanitize-html` for HTML/XSS prevention and `class-transformer` decorators for field cleaning
- Always use parameterized queries (via ORM) instead of string concatenation to prevent SQL injection
- Sanitize user-generated content before storage, not at display time

## Code Examples

**Sanitizing user input to prevent XSS attacks using class-sanitizer**

```typescript
import * as sanitizeHtml from 'sanitize-html';
import { Transform } from 'class-transformer';

export function SanitizeHtml() {
  return Transform(({ value }) => {
    if (typeof value !== 'string') return value;
    return sanitizeHtml(value, {
      allowedTags: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
      allowedAttributes: {
        a: ['href', 'title'],
      },
      allowedSchemes: ['http', 'https'],
    });
  });
}

export class CreatePostDto {
  @IsString()
  @MaxLength(100)
  title: string;

  @IsString()
  @MaxLength(10000)
  @SanitizeHtml()
  content: string;
}
```


## Resources

- [Security Overview](https://docs.nestjs.com/security/overview) — NestJS security best practices

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*