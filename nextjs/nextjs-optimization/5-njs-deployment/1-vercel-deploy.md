---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-vercel-deploy"
---

# Deploying to Vercel

Vercel is the company behind Next.js and offers the most optimized deployment experience.

## Quick Deploy

1. Push your code to GitHub/GitLab/Bitbucket.
2. Import the repository on [vercel.com](https://vercel.com).
3. Vercel auto-detects Next.js and configures everything.

## Features

- **Automatic deployments**: Every push triggers a build.
- **Preview deployments**: Every PR gets a unique URL.
- **Edge Network**: Global CDN with edge caching.
- **Serverless Functions**: Route Handlers and Server Actions.
- **Analytics**: Built-in performance monitoring.

## Environment Variables

Set them in the Vercel dashboard:
- `DATABASE_URL`
- `NEXTAUTH_SECRET`
- `API_KEY`

For local development, use `.env.local` (not committed to git).

## Build Command

Vercel runs:
```bash
next build
```

Output is automatically optimized for the Vercel platform.

## Resources

- [Deploying to Vercel](https://nextjs.org/docs/app/building-your-application/deploying) — Official deployment guide

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*