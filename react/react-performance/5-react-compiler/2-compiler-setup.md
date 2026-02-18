---
source_course: "react-performance"
source_lesson: "react-performance-compiler-setup"
---

# Setting Up React Compiler

Integrate the compiler into your build.

## Installation

```bash
npm install babel-plugin-react-compiler
```

## Babel Configuration

```js
// babel.config.js
module.exports = {
  plugins: [
    ['babel-plugin-react-compiler', {
      // Configuration options
    }],
  ],
};
```

## Vite Setup

```js
// vite.config.js
import react from '@vitejs/plugin-react';

export default {
  plugins: [
    react({
      babel: {
        plugins: ['babel-plugin-react-compiler'],
      },
    }),
  ],
};
```

## Next.js Setup

```js
// next.config.js
module.exports = {
  experimental: {
    reactCompiler: true,
  },
};
```

## Gradual Adoption

### Opt-in Mode

```js
// babel.config.js
module.exports = {
  plugins: [
    ['babel-plugin-react-compiler', {
      compilationMode: 'annotation', // Only compile annotated
    }],
  ],
};
```

```jsx
// Use directive to opt-in
'use memo';

function MyComponent() {
  // This component will be compiled
}
```

### Opt-out Mode

```jsx
// Skip compilation for specific components
'use no memo';

function LegacyComponent() {
  // Won't be compiled
}
```

## ESLint Plugin

```bash
npm install eslint-plugin-react-compiler
```

```js
// .eslintrc.js
module.exports = {
  plugins: ['react-compiler'],
  rules: {
    'react-compiler/react-compiler': 'error',
  },
};
```

The ESLint plugin catches code patterns that break the compiler.

## Verifying Compilation

```jsx
// Check if component was compiled
function MyComponent() {
  // Add this in development
  if (process.env.NODE_ENV === 'development') {
    console.log('Compiled:', MyComponent._compiled);
  }
  return <div>...</div>;
}
```

## Debugging

```js
// babel.config.js
module.exports = {
  plugins: [
    ['babel-plugin-react-compiler', {
      // Log compilation details
      logger: {
        logEvent(event) {
          console.log(event);
        },
      },
    }],
  ],
};
```

---

> 📘 *This lesson is part of the [React Performance Deep Dive](https://stanza.dev/courses/react-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*