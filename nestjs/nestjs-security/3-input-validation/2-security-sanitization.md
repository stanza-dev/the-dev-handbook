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

```bash
npm install sanitize-html
npm install -D @types/sanitize-html
```

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

### Strip HTML Completely

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

### Trim and Normalize

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

### Path Sanitization

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

### SQL Injection Prevention

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

Sanitization cleans input to remove dangerous content. Use sanitize-html for HTML content, class-transformer for field transformations, and parameterized queries for database operations. Sanitize before storage, not display.

## Resources

- [Security Overview](https://docs.nestjs.com/security/overview) — NestJS security best practices

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*