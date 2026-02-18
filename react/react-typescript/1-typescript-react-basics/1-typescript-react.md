---
source_course: "react-typescript"
source_lesson: "react-typescript-why-typescript-react"
---

# Why TypeScript for React?

TypeScript adds static type checking to JavaScript, catching errors before runtime.

## The Problems TypeScript Solves

### 1. Prop Errors at Compile Time

```jsx
// JavaScript - Error only at runtime
<UserCard naem="John" /> // Typo in 'name'

// TypeScript - Error at compile time
<UserCard naem="John" />
// Error: Property 'naem' does not exist. Did you mean 'name'?
```

### 2. Autocomplete & IntelliSense

TypeScript provides:
- Props suggestions
- Method signatures
- Documentation on hover
- Refactoring support

### 3. Self-Documenting Code

```tsx
// Types ARE documentation
type UserCardProps = {
  name: string;
  email: string;
  role: 'admin' | 'user' | 'guest';
  onSelect?: (id: string) => void;
};
```

### 4. Safer Refactoring

Rename a prop? TypeScript shows you every place it's used.

## Setting Up

```bash
# Create React App with TypeScript
npx create-react-app my-app --template typescript

# Or add to existing project
npm install typescript @types/react @types/react-dom
```

## File Extensions

- `.ts` - TypeScript files (no JSX)
- `.tsx` - TypeScript files with JSX

## tsconfig.json Basics

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["DOM", "DOM.Iterable", "ES2020"],
    "jsx": "react-jsx",
    "strict": true,
    "moduleResolution": "bundler"
  }
}
```

## Type vs Interface

Both work for React props. Use types for consistency:

```tsx
// Preferred: Type alias
type ButtonProps = {
  label: string;
  onClick: () => void;
};

// Also valid: Interface
interface ButtonProps {
  label: string;
  onClick: () => void;
}
```

📚 **Learn more**: [Using TypeScript](https://react.dev/learn/typescript)

---

> 📘 *This lesson is part of the [React TypeScript Mastery](https://stanza.dev/courses/react-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*