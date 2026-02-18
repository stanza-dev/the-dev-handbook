---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-nextauth-database"
---

# Database Adapters

## Introduction

By default, Auth.js uses JWTs—no database needed. But what if you want to store users, manage sessions server-side, or link multiple OAuth accounts? Database adapters connect Auth.js to your database.

## Key Concepts

**Adapters** handle database operations for Auth.js:

- **User storage**: Create and retrieve user records
- **Account linking**: Associate OAuth accounts with users
- **Session storage**: Store sessions in the database (instead of JWTs)
- **Verification tokens**: For email magic links

## Real World Context

Use a database adapter when:
- You need to store additional user data
- You want server-side sessions (revocable, consistent across devices)
- Users can link multiple OAuth providers to one account
- You're using email magic link authentication

## Deep Dive

### Installing Prisma Adapter

```bash
npm install @auth/prisma-adapter
```

### Prisma Schema

```prisma
// prisma/schema.prisma
model User {
  id            String    @id @default(cuid())
  name          String?
  email         String?   @unique
  emailVerified DateTime?
  image         String?
  role          String    @default("user")
  accounts      Account[]
  sessions      Session[]
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
}

model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String? @db.Text
  access_token      String? @db.Text
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String? @db.Text
  session_state     String?
  user              User    @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime

  @@unique([identifier, token])
}
```

### Auth Configuration with Adapter

```typescript
// auth.ts
import NextAuth from 'next-auth';
import { PrismaAdapter } from '@auth/prisma-adapter';
import { prisma } from '@/lib/prisma';
import GitHub from 'next-auth/providers/github';
import Google from 'next-auth/providers/google';

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: PrismaAdapter(prisma),
  providers: [
    GitHub,
    Google,
  ],
  session: {
    strategy: 'database', // Use database sessions
  },
});
```

### JWT vs Database Sessions

```typescript
// JWT sessions (default) - stored in cookie
session: {
  strategy: 'jwt',
  maxAge: 30 * 24 * 60 * 60, // 30 days
}

// Database sessions - stored in DB, revocable
session: {
  strategy: 'database',
  maxAge: 30 * 24 * 60 * 60,
  updateAge: 24 * 60 * 60, // Update session every 24 hours
}
```

### Account Linking

With a database adapter, users can link multiple providers:

```typescript
// User signs in with Google first → creates user
// Same user signs in with GitHub later → links to existing user (by email)

callbacks: {
  async signIn({ user, account }) {
    // Check if user exists with this email
    const existingUser = await prisma.user.findUnique({
      where: { email: user.email! },
      include: { accounts: true },
    });

    if (existingUser) {
      // Check if this provider is already linked
      const linkedAccount = existingUser.accounts.find(
        (a) => a.provider === account?.provider
      );

      if (!linkedAccount) {
        // Link new provider to existing user
        // Auth.js does this automatically with an adapter
      }
    }

    return true;
  },
}
```

### Available Adapters

```bash
# Official adapters
npm install @auth/prisma-adapter    # Prisma
npm install @auth/drizzle-adapter   # Drizzle ORM
npm install @auth/mongodb-adapter   # MongoDB
npm install @auth/supabase-adapter  # Supabase
npm install @auth/fauna-adapter     # Fauna
npm install @auth/neo4j-adapter     # Neo4j
```

## Common Pitfalls

1. **Schema mismatch**: The adapter expects specific table/column names. Use the official schema or configure field mapping.

2. **JWT callbacks with database sessions**: When using database sessions, the `jwt` callback doesn't run. Use the `session` callback instead.

3. **Performance with database sessions**: Every page load queries the database. Use caching or consider JWTs for high-traffic sites.

## Best Practices

- **Start with JWTs**: Simpler to set up, works without a database
- **Add adapter when needed**: When you need account linking, user storage, or revocable sessions
- **Use the official schema**: Don't customize table names unless you have to
- **Monitor performance**: Database sessions add latency—ensure your DB is optimized

## Summary

Database adapters connect Auth.js to your database for user storage, account linking, and server-side sessions. Install an adapter, add the required schema, and configure `session.strategy: 'database'`. Choose JWTs for simplicity and performance, database sessions for revocability and complex user data.

## Resources

- [Auth.js Adapters](https://authjs.dev/getting-started/adapters) — Database adapter documentation
- [Prisma Adapter](https://authjs.dev/getting-started/adapters/prisma) — Prisma adapter setup guide

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*