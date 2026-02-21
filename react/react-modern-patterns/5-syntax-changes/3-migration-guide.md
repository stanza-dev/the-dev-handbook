---
source_course: "react-modern-patterns"
source_lesson: "react-19-migration-guide"
---

# React 19 Migration Guide

## Introduction

Upgrading to React 19 is designed to be smooth, but there are breaking changes you need to be aware of. This lesson covers the key changes beyond the ones we have already discussed (ref as prop, context as provider), including changes to Strict Mode, removed APIs, and new defaults. Having a clear migration plan prevents surprises and ensures your team can adopt React 19's benefits incrementally.

## Key Concepts

- **Removed legacy APIs**: `propTypes`, `defaultProps` on function components, string refs, `ReactDOM.render`, and legacy context are removed.
- **Strict Mode changes**: In React 19, Strict Mode double-invokes effects only once during development (previously it was twice in Strict Mode for the initial mount).
- **Ref callbacks cleanup**: Ref callbacks can now return cleanup functions, and React will call the cleanup when the ref is detached.
- **Codemods**: Official codemods automate many of the migration steps, including replacing `forwardRef`, updating context patterns, and removing deprecated APIs.

## Real World Context

A team maintaining a large application with 500+ components might have dozens of `forwardRef` usages, some legacy `defaultProps`, and a few components using string refs from early React days. Rather than manually updating every file, the team can run the official codemods to automate the repetitive changes, then manually review the handful of cases that need human judgment.

## Deep Dive

Here are the key breaking changes and how to handle them:

**1. defaultProps removed on function components:**

```tsx
// Before — no longer works in React 19
function Button({ size }) { /* ... */ }
Button.defaultProps = { size: "md" };

// After — use JavaScript default parameters
function Button({ size = "md" }) { /* ... */ }
```

**2. String refs removed:**

```tsx
// Before — removed in React 19
class Input extends React.Component {
  render() {
    return <input ref="myInput" />;  // Error in React 19
  }
}

// After — use createRef or useRef
class Input extends React.Component {
  inputRef = React.createRef();
  render() {
    return <input ref={this.inputRef} />;
  }
}
```

**3. `ReactDOM.render` removed — use `createRoot`:**

```tsx
// Before
import ReactDOM from "react-dom";
ReactDOM.render(<App />, document.getElementById("root"));

// After
import { createRoot } from "react-dom/client";
const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

**4. Running the codemods:**

```bash
# Replace forwardRef with ref prop
npx codemod@latest react/19/replace-reactdom-render

# Replace string refs
npx codemod@latest react/19/replace-string-ref

# Remove defaultProps
npx codemod@latest react/19/replace-use-form-state
```

**5. New JSX transform is required:**

React 19 requires the new JSX transform (introduced in React 17). If your project still uses `import React from "react"` at the top of every file, you may need to update your build configuration.

## Common Pitfalls

1. **Not running codemods before manual migration** — Codemods handle the mechanical changes (forwardRef, defaultProps, string refs) automatically. Running them first reduces the manual work dramatically.
2. **Ignoring TypeScript errors after upgrade** — React 19's type definitions have changed. Run `tsc --noEmit` after upgrading to catch type errors early, especially around ref types and event handlers.

## Best Practices

1. **Upgrade incrementally** — Update React to 19, fix breaking changes one category at a time (refs, then context, then removed APIs), and run tests between each step.
2. **Test with Strict Mode enabled** — React 19 changes Strict Mode behavior. Keep it enabled during development to catch issues early.

## Summary

- React 19 removes `defaultProps` on function components, string refs, `ReactDOM.render`, and legacy context — use modern equivalents.
- Official codemods automate most migration tasks; run them before doing manual changes.
- The new JSX transform is required, and TypeScript definitions have been updated — check types after upgrading.

## Code Examples

**Key React 19 migration patterns in one file**

```tsx
// Migration checklist in code:

// 1. defaultProps -> default parameters
function Card({ variant = "outlined" }: { variant?: string }) {
  return <div className={`card-${variant}`}>Content</div>;
}

// 2. forwardRef -> ref prop
function Input({ ref, ...props }: { ref?: React.Ref<HTMLInputElement> }) {
  return <input ref={ref} {...props} />;
}

// 3. Context.Provider -> Context
import { createContext } from "react";
const Theme = createContext("light");
function App({ children }: { children: React.ReactNode }) {
  return <Theme value="dark">{children}</Theme>;
}

// 4. createRoot (already required since React 18)
import { createRoot } from "react-dom/client";
createRoot(document.getElementById("root")!).render(<App />);
```


## Resources

- [React 19 Upgrade Guide](https://react.dev/blog/2024/04/25/react-19-upgrade-guide) — Complete official upgrade guide for React 19

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*