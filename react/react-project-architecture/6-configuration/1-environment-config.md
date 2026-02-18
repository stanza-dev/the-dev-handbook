---
source_course: "react-project-architecture"
source_lesson: "react-project-architecture-environment-config"
---

# Environment Configuration

Manage different environments safely.

## Environment Files

```
.env                  # Default/shared
.env.local            # Local overrides (gitignored)
.env.development      # Development
.env.production       # Production
.env.test             # Testing
```

## Vite Environment Variables

```bash
# .env
VITE_API_URL=https://api.example.com
VITE_APP_TITLE=My App
VITE_FEATURE_NEW_UI=true
```

```tsx
// Access in code
const apiUrl = import.meta.env.VITE_API_URL;
const appTitle = import.meta.env.VITE_APP_TITLE;
```

## Type-Safe Config

```tsx
// config/env.ts
import { z } from 'zod';

const envSchema = z.object({
  VITE_API_URL: z.string().url(),
  VITE_APP_TITLE: z.string(),
  VITE_FEATURE_NEW_UI: z.string().transform(v => v === 'true'),
});

const parsed = envSchema.safeParse(import.meta.env);

if (!parsed.success) {
  console.error('Invalid environment variables:', parsed.error.flatten());
  throw new Error('Invalid environment configuration');
}

export const env = parsed.data;
```

## App Configuration

```tsx
// config/index.ts
import { env } from './env';

export const config = {
  api: {
    baseUrl: env.VITE_API_URL,
    timeout: 30000,
  },
  app: {
    title: env.VITE_APP_TITLE,
    version: '1.0.0',
  },
  features: {
    newUI: env.VITE_FEATURE_NEW_UI,
  },
} as const;
```

## Feature Flags

```tsx
// config/features.ts
export const features = {
  newDashboard: env.VITE_FEATURE_NEW_DASHBOARD,
  darkMode: true,
  betaFeatures: env.MODE === 'development',
} as const;

// hooks/useFeature.ts
export function useFeature(feature: keyof typeof features) {
  return features[feature];
}

// Usage
function Dashboard() {
  const hasNewDashboard = useFeature('newDashboard');
  
  if (hasNewDashboard) {
    return <NewDashboard />;
  }
  return <OldDashboard />;
}
```

## Runtime Configuration

```tsx
// For values that can change without rebuild
// public/config.json
{
  "maintenanceMode": false,
  "announcement": "Welcome to our new site!"
}

// hooks/useRuntimeConfig.ts
export function useRuntimeConfig() {
  return useQuery({
    queryKey: ['runtime-config'],
    queryFn: () => fetch('/config.json').then(r => r.json()),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
}
```

## Multi-Environment Setup

```bash
# package.json scripts
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "build:staging": "vite build --mode staging",
    "build:production": "vite build --mode production"
  }
}
```

```bash
# .env.staging
VITE_API_URL=https://staging-api.example.com
VITE_ENV=staging
```

---

> 📘 *This lesson is part of the [React Project Architecture](https://stanza.dev/courses/react-project-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*