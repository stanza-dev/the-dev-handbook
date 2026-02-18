---
source_course: "react-server-components"
source_lesson: "react-server-components-rules-of-react"
---

# The Rules of React

The React Compiler enforces rules that have always been part of React's mental model. Understanding these rules is crucial for writing correct React code.

## Rule 1: Components and Hooks Must Be Pure

During rendering, components should only:
- Read props, state, and context
- Return JSX

```tsx
// ❌ Side effects during render
function BadComponent() {
  document.title = 'Updated!'; // Side effect!
  localStorage.setItem('key', 'value'); // Side effect!
  
  return <div>Content</div>;
}

// ✅ Side effects in event handlers or effects
function GoodComponent() {
  useEffect(() => {
    document.title = 'Updated!';
  }, []);
  
  const handleClick = () => {
    localStorage.setItem('key', 'value');
  };
  
  return <button onClick={handleClick}>Save</button>;
}
```

## Rule 2: React Calls Components and Hooks

Don't call components as functions or hooks outside components:

```tsx
// ❌ Calling component as function
function Parent() {
  const result = Child(); // Wrong!
  return <div>{result}</div>;
}

// ✅ Rendering as JSX
function Parent() {
  return <div><Child /></div>;
}

// ❌ Calling hooks outside components
const value = useState(0); // Wrong!

// ✅ Hooks inside components
function Component() {
  const [value, setValue] = useState(0);
}
```

## Rule 3: Rules of Hooks

Hooks must be called:
- At the top level of components
- Not inside conditions, loops, or nested functions

```tsx
// ❌ Conditional hook
function Component({ condition }) {
  if (condition) {
    const [state, setState] = useState(0); // Wrong!
  }
}

// ✅ Unconditional hook, conditional usage
function Component({ condition }) {
  const [state, setState] = useState(0);
  
  if (condition) {
    // Use state here
  }
}
```

## Rule 4: Only Call Hooks from React Functions

Hooks can only be called from:
- Function components
- Custom hooks

```tsx
// ❌ Hook in regular function
function helper() {
  const [state, setState] = useState(0); // Wrong!
}

// ✅ Hook in custom hook
function useHelper() {
  const [state, setState] = useState(0); // Correct!
  return state;
}
```

## Why These Rules Matter

### 1. Predictable Rendering

React may render components multiple times, skip renders, or render in different orders. Pure components work correctly regardless.

### 2. Concurrent Features

React's concurrent features (Suspense, transitions) rely on being able to pause and resume rendering. Side effects during render break this.

### 3. Compiler Optimization

The React Compiler assumes these rules are followed. Violations can cause incorrect optimizations.

## Detecting Violations

### Strict Mode

```tsx
<StrictMode>
  <App />
</StrictMode>
```

Strict Mode helps detect:
- Impure renders (by rendering twice)
- Missing effect cleanup
- Deprecated APIs

### ESLint Rules

```js
{
  "extends": [
    "plugin:react-hooks/recommended"
  ]
}
```

Catches:
- Hooks called conditionally
- Missing dependencies
- Invalid hook calls

## Resources

- [Rules of React](https://react.dev/reference/rules) — Official documentation on the Rules of React
- [Keeping Components Pure](https://react.dev/learn/keeping-components-pure) — Guide to writing pure React components

---

> 📘 *This lesson is part of the [React Server Components](https://stanza.dev/courses/react-server-components) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*