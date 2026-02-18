---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-env-variables"
---

# Environment Variables Best Practices

## Introduction

Environment variables configure your app for different deployments. Mishandling them leads to security issues, broken builds, and frustrated developers.

## Key Concepts

**Variable types**:

- `NEXT_PUBLIC_*`: Exposed to browser (public)
- Regular variables: Server-only (private)
- Build-time vs runtime: When they're read

## Deep Dive

### Server vs Client Variables

```bash
# .env.local

# Private - only available on server
DATABASE_URL="postgresql://..."
STRIPE_SECRET_KEY="sk_live_..."

# Public - exposed to browser
NEXT_PUBLIC_API_URL="https://api.example.com"
NEXT_PUBLIC_ANALYTICS_ID="UA-123456"
```

### Validation at Startup

```typescript
// lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  STRIPE_SECRET_KEY: z.string().startsWith('sk_'),
  NEXT_PUBLIC_API_URL: z.string().url(),
});

export const env = envSchema.parse(process.env);
```

### Per-Environment Files

```
.env                # Default
.env.local          # Local overrides (gitignored)
.env.development    # Development defaults
.env.production     # Production defaults
```

## Summary

Prefix client-side variables with NEXT_PUBLIC_, validate variables at startup, and use .env.local for secrets. Never commit secrets to git.

## Resources

- [Environment Variables](https://nextjs.org/docs/app/building-your-application/configuring/environment-variables) — Environment variable configuration

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*