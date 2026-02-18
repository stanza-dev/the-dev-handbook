---
source_course: "react"
source_lesson: "react-setting-up-react"
---

# Setting Up a React Project

The recommended way to start a new React project in 2024+ is using a framework.

## Recommended Frameworks

### Next.js (Most Popular)
```bash
npx create-next-app@latest my-app
```

### Vite (Lightweight)
```bash
npm create vite@latest my-app -- --template react-ts
```

## Project Structure

A typical React project includes:
- `src/` - Your application code
- `public/` - Static assets
- `package.json` - Dependencies and scripts
- `tsconfig.json` - TypeScript configuration (if using TS)

## Code Examples

**Creating a React project with Vite**

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


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*