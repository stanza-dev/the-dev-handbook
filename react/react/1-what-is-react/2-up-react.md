---
source_course: "react"
source_lesson: "react-setting-up-react"
---

# Setting Up a React Project

## Introduction

Before you can write your first component, you need a development environment that can compile JSX, bundle modules, and serve your application locally. This lesson walks you through the two most popular ways to bootstrap a React project in 2024 and beyond, so you can start coding in minutes.

## Key Concepts

- **Bundler**: A tool (Vite, Webpack, Turbopack) that takes your source files and produces optimized bundles for the browser.
- **Framework vs. Library Setup**: A framework like Next.js includes routing, server rendering, and build tooling. A plain library setup (Vite + React) gives you just the view layer.
- **TypeScript Template**: Most modern React projects use TypeScript for type safety. Templates scaffold the tsconfig and type declarations for you.
- **Development Server**: A local server with hot module replacement (HMR) that reflects code changes instantly without a full page reload.

## Real World Context

On a professional team, you will rarely set up a project from absolute scratch. Instead, you will clone a repository or run a scaffolding command. Knowing what each generated file does, however, is essential. When the build breaks or you need to add a custom plugin, understanding the project structure lets you diagnose problems quickly instead of guessing.

## Deep Dive

The React team officially recommends starting with a framework. Here are the two most common paths:

### Next.js (Recommended for Production)

```bash
npx create-next-app@latest my-app --typescript
cd my-app
npm run dev
```

Next.js gives you file-based routing, server-side rendering, static generation, and API routes out of the box. Your development server starts at `http://localhost:3000`.

### Vite (Lightweight, Client-Only)

```bash
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install
npm run dev
```

Vite is blazing fast for development and produces highly optimized production builds. Your dev server starts at `http://localhost:5173`.

After scaffolding, a typical project structure looks like this:

```
my-app/
  src/
    App.tsx          # Root component
    main.tsx         # Entry point
  public/            # Static assets
  index.html         # HTML shell
  package.json       # Dependencies and scripts
  tsconfig.json      # TypeScript configuration
  vite.config.ts     # Bundler configuration
```

The `src/main.tsx` file is the entry point where React mounts into the DOM:

```tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

`StrictMode` is a development-only wrapper that helps you find common bugs by intentionally double-invoking components during development.

## Common Pitfalls

1. **Skipping TypeScript** — Starting without TypeScript seems faster, but migrating later is painful. Always use the TypeScript template from day one.
2. **Confusing dev and production builds** — The dev server includes extra warnings and source maps. Always test with `npm run build && npm run preview` before deploying.

## Best Practices

1. **Use `StrictMode`** — Keep it enabled during development to catch impure renders, missing cleanup functions, and deprecated APIs early.
2. **Choose a framework for new projects** — Unless you have a specific reason for a client-only SPA, prefer Next.js for its built-in optimizations and server capabilities.

## Summary

- Use `create-next-app` for full-featured projects or Vite for lightweight client-only apps.
- The entry point (`main.tsx`) mounts your root component into the DOM using `createRoot`.
- Always enable `StrictMode` during development and start with a TypeScript template.

## Code Examples

**Creating a React project with Vite and TypeScript**

```bash
# Create a new React app with Vite and TypeScript
npm create vite@latest my-react-app -- --template react-ts

# Navigate to project
cd my-react-app

# Install dependencies
npm install

# Start development server
npm run dev
```


## Resources

- [Start a New React Project](https://react.dev/learn/start-a-new-react-project) — Official guide on starting a new React project
- [Vite Getting Started](https://vite.dev/guide/) — Vite official getting started guide

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*