---
source_course: "vue-typescript"
source_lesson: "vue-typescript-why-typescript-vue"
---

# Why TypeScript with Vue?

TypeScript adds static typing to JavaScript, catching errors at compile time rather than runtime. Vue 3 was rewritten in TypeScript and provides excellent TS support.

## Benefits in Vue Projects

### 1. Catch Errors Early

```typescript
// JavaScript - runtime error
const user = { name: 'Alice', age: 25 }
console.log(user.email)  // undefined, no error until runtime

// TypeScript - compile-time error
type User = { name: string; age: number }
const user: User = { name: 'Alice', age: 25 }
console.log(user.email)  // ❌ Error: Property 'email' does not exist
```

### 2. Better IDE Support

- Intelligent autocomplete
- Inline documentation
- Refactoring tools
- Go to definition
- Find all references

### 3. Self-Documenting Code

```typescript
// Types serve as documentation
type Product = {
  id: number
  name: string
  price: number
  category: 'electronics' | 'clothing' | 'food'
  inStock: boolean
}

function formatPrice(product: Product): string {
  return `$${product.price.toFixed(2)}`
}
```

### 4. Safer Refactoring

```typescript
// Rename 'name' to 'title' - TypeScript finds all usages
type Post = {
  title: string  // Changed from 'name'
  content: string
}

// All usages of post.name will show errors
```

## Vue's TypeScript Support

Vue 3 provides:
- Full type inference in templates
- Typed props, emits, slots
- Typed refs and reactive
- Typed provide/inject
- Generic components

## Project Setup

### Create New Project

```bash
npm create vue@latest my-app
# Select: TypeScript, Vue Router, Pinia, etc.
```

### Manual Setup

```bash
npm install -D typescript vue-tsc @types/node
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "jsx": "preserve",
    "jsxImportSource": "vue",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "esModuleInterop": true,
    "lib": ["ESNext", "DOM"],
    "skipLibCheck": true,
    "noEmit": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*.ts", "src/**/*.tsx", "src/**/*.vue"],
  "exclude": ["node_modules"]
}
```

### Vite Config

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()]
})
```

## IDE Setup

### VS Code / Cursor

Install extensions:
- **Vue - Official** (formerly Volar) - Vue language support
- **TypeScript Vue Plugin** - TS support in .vue files

### Takeover Mode (Optional)

For better performance, disable built-in TS extension for .vue files:

1. Open Command Palette
2. "Extensions: Show Built-in Extensions"
3. Find "TypeScript and JavaScript Language Features"
4. Right-click → "Disable (Workspace)"

## Type Checking

```bash
# Check types without building
npx vue-tsc --noEmit

# In package.json scripts
{
  "scripts": {
    "type-check": "vue-tsc --noEmit",
    "build": "vue-tsc --noEmit && vite build"
  }
}
```

## Resources

- [TypeScript with Vue](https://vuejs.org/guide/typescript/overview.html) — Official guide on using TypeScript with Vue

---

> 📘 *This lesson is part of the [TypeScript with Vue](https://stanza.dev/courses/vue-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*