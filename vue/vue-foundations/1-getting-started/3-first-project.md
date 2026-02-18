---
source_course: "vue-foundations"
source_lesson: "vue-foundations-creating-first-project"
---

# Creating Your First Vue Project

Let's create your first Vue project using the official scaffolding tool `create-vue`. This tool sets up a modern Vue project with Vite, giving you an incredibly fast development experience.

## Creating a New Project

Open your terminal and run:

```bash
npm create vue@latest
```

Or with other package managers:

```bash
# With pnpm
pnpm create vue@latest

# With yarn
yarn create vue

# With bun
bun create vue@latest
```

## Project Configuration

The tool will ask you several questions. For your first project, we recommend:

```
✔ Project name: my-vue-app
✔ Add TypeScript? Yes
✔ Add JSX Support? No
✔ Add Vue Router? No (we'll add it later)
✔ Add Pinia? No (we'll add it later)
✔ Add Vitest? No
✔ Add an End-to-End Testing Solution? No
✔ Add ESLint for code quality? Yes
✔ Add Prettier for code formatting? Yes
✔ Add Vue DevTools extension? Yes
```

## Install Dependencies and Start

```bash
cd my-vue-app
npm install
npm run dev
```

You'll see:

```
  VITE v5.x.x  ready in 234 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

Open **http://localhost:5173** in your browser to see your Vue app running!

## Project Structure

Your project has this structure:

```
my-vue-app/
├── node_modules/        # Dependencies
├── public/              # Static assets (served as-is)
│   └── favicon.ico
├── src/                 # Source code
│   ├── assets/          # Processed assets (images, styles)
│   ├── components/      # Vue components
│   │   └── HelloWorld.vue
│   ├── App.vue          # Root component
│   └── main.ts          # Application entry point
├── index.html           # HTML entry point
├── package.json         # Dependencies and scripts
├── tsconfig.json        # TypeScript configuration
├── vite.config.ts       # Vite configuration
└── .eslintrc.cjs        # ESLint configuration
```

## Understanding Key Files

### index.html

The HTML entry point that loads your Vue app:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" href="/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite App</title>
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

The `<div id="app">` is where Vue mounts your application.

### src/main.ts

The JavaScript entry point that creates and mounts the Vue app:

```typescript
import { createApp } from 'vue'
import App from './App.vue'

import './assets/main.css'

createApp(App).mount('#app')
```

This code:
1. Imports `createApp` from Vue
2. Imports your root component (`App.vue`)
3. Creates a Vue application instance
4. Mounts it to the `#app` element

### src/App.vue

The root component of your application:

```vue
<script setup lang="ts">
import HelloWorld from './components/HelloWorld.vue'
</script>

<template>
  <header>
    <div class="wrapper">
      <HelloWorld msg="You did it!" />
    </div>
  </header>
</template>

<style scoped>
header {
  line-height: 1.5;
}
</style>
```

## Modifying Your First Component

Let's edit `App.vue` to create something of our own:

```vue
<script setup lang="ts">
import { ref } from 'vue'

const message = ref('Hello, Vue!')
const count = ref(0)

function increment() {
  count.value++
}
</script>

<template>
  <div class="app">
    <h1>{{ message }}</h1>
    <p>Count: {{ count }}</p>
    <button @click="increment">Increment</button>
  </div>
</template>

<style scoped>
.app {
  text-align: center;
  padding: 2rem;
}

button {
  background: #42b883;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background: #3aa876;
}
</style>
```

Save the file and watch your browser automatically update—this is **Hot Module Replacement (HMR)** in action!

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Lint code
npm run lint

# Format code
npm run format
```

## Resources

- [Creating a Vue Application](https://vuejs.org/guide/essentials/application.html) — Official documentation on creating and configuring Vue applications

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*