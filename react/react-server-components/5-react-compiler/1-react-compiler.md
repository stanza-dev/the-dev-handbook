---
source_course: "react-server-components"
source_lesson: "react-server-components-react-compiler"
---

# React Compiler: Automatic Optimization

The React Compiler (formerly "React Forget") is a build-time tool that automatically optimizes your React code by adding memoization.

## What It Does

The compiler automatically inserts `useMemo`, `useCallback`, and `memo` where beneficial:

```tsx
// Your code
function ProductList({ products, onSelect }) {
  const sorted = products.sort((a, b) => a.price - b.price);
  
  return (
    <ul>
      {sorted.map(p => (
        <ProductItem 
          key={p.id} 
          product={p} 
          onSelect={() => onSelect(p.id)}
        />
      ))}
    </ul>
  );
}

// What the compiler produces (conceptually)
function ProductList({ products, onSelect }) {
  const sorted = useMemo(
    () => products.sort((a, b) => a.price - b.price),
    [products]
  );
  
  return (
    <ul>
      {sorted.map(p => (
        <ProductItem 
          key={p.id} 
          product={p} 
          onSelect={useCallback(() => onSelect(p.id), [onSelect, p.id])}
        />
      ))}
    </ul>
  );
}
```

## Requirements

The compiler requires your code to follow the **Rules of React**:

### 1. Components Must Be Pure

```tsx
// ❌ Impure - modifies external state during render
let count = 0;
function Counter() {
  count++; // Side effect during render!
  return <div>{count}</div>;
}

// ✅ Pure - no side effects during render
function Counter({ count }) {
  return <div>{count}</div>;
}
```

### 2. Props and State Are Immutable

```tsx
// ❌ Mutating props
function List({ items }) {
  items.push(newItem); // Mutating!
  return <ul>{items.map(...)}</ul>;
}

// ✅ Creating new arrays
function List({ items }) {
  const allItems = [...items, newItem];
  return <ul>{allItems.map(...)}</ul>;
}
```

### 3. Hook Return Values Are Immutable

```tsx
// ❌ Mutating state directly
function Form() {
  const [user, setUser] = useState({ name: '' });
  
  const handleChange = (e) => {
    user.name = e.target.value; // Mutating!
    setUser(user);
  };
}

// ✅ Creating new objects
function Form() {
  const [user, setUser] = useState({ name: '' });
  
  const handleChange = (e) => {
    setUser({ ...user, name: e.target.value });
  };
}
```

## Enabling the Compiler

### Next.js 15+

```js
// next.config.js
module.exports = {
  experimental: {
    reactCompiler: true
  }
};
```

### Babel Plugin

```js
// babel.config.js
module.exports = {
  plugins: [
    ['babel-plugin-react-compiler', {
      // options
    }]
  ]
};
```

## ESLint Plugin

The compiler comes with an ESLint plugin to catch violations:

```js
// eslint.config.js
import reactCompiler from 'eslint-plugin-react-compiler';

export default [
  {
    plugins: {
      'react-compiler': reactCompiler
    },
    rules: {
      'react-compiler/react-compiler': 'error'
    }
  }
];
```

## When Manual Optimization Is Still Needed

The compiler handles most cases, but you might still need manual optimization for:

1. **Complex memoization logic** with custom comparison
2. **Refs** that need stable identity
3. **External library integration** with specific requirements

## Resources

- [React Compiler](https://react.dev/learn/react-compiler) — Official React documentation on the React Compiler

---

> 📘 *This lesson is part of the [React Server Components](https://stanza.dev/courses/react-server-components) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*