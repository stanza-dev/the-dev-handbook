---
source_course: "react-intermediate"
source_lesson: "react-css-in-js-patterns"
---

# CSS-in-JS Patterns

## Introduction

Beyond basic component styling, CSS-in-JS libraries like Styled Components offer powerful patterns for building maintainable, scalable style systems. From global styles and CSS resets to animation keyframes and media query helpers, these patterns help you organize your styles effectively as your application grows. This lesson covers the most important production patterns you will use with CSS-in-JS.

## Key Concepts

- **Global styles**: `createGlobalStyle` creates a component that injects global CSS (resets, font imports, body styles) when rendered.
- **Keyframe animations**: The `keyframes` helper generates unique animation names, preventing collisions between animations in different components.
- **CSS helper**: The `css` helper lets you define reusable CSS fragments that can be interpolated into multiple styled components.
- **`as` prop**: Every styled component accepts an `as` prop that changes the rendered HTML element while keeping the styles.

## Real World Context

A large SaaS application needs a consistent CSS reset across all pages, shared animation keyframes for loading spinners and transitions, a set of reusable media query mixins for responsive design, and the ability to render a styled Button as a link (`<a>`) when it navigates to a URL. CSS-in-JS patterns handle all of these needs within the JavaScript-first paradigm.

## Deep Dive

**Global styles:**

```tsx
import { createGlobalStyle } from "styled-components";

const GlobalStyle = createGlobalStyle`
  *, *::before, *::after {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  body {
    font-family: "Inter", -apple-system, sans-serif;
    line-height: 1.6;
    color: ${(props) => props.theme.text};
    background: ${(props) => props.theme.background};
  }

  a {
    color: inherit;
    text-decoration: none;
  }
`;

function App() {
  return (
    <>
      <GlobalStyle />
      <RestOfApp />
    </>
  );
}
```

**Keyframe animations:**

```tsx
import styled, { keyframes } from "styled-components";

const spin = keyframes`
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
`;

const fadeIn = keyframes`
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
`;

const Spinner = styled.div`
  width: 24px;
  height: 24px;
  border: 3px solid #e5e7eb;
  border-top-color: #2563eb;
  border-radius: 50%;
  animation: ${spin} 0.8s linear infinite;
`;

const FadeInCard = styled.div`
  animation: ${fadeIn} 0.3s ease-out;
  padding: 16px;
  border-radius: 8px;
`;
```

**Reusable CSS fragments:**

```tsx
import styled, { css } from "styled-components";

const truncate = css`
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
`;

const flexCenter = css`
  display: flex;
  align-items: center;
  justify-content: center;
`;

const Title = styled.h2`
  ${truncate}
  font-size: 18px;
  max-width: 200px;
`;

const CenteredContainer = styled.div`
  ${flexCenter}
  min-height: 100vh;
`;
```

**The `as` prop for polymorphism:**

```tsx
const Button = styled.button`
  padding: 10px 20px;
  background: #2563eb;
  color: white;
  border: none;
  border-radius: 6px;
`;

// Render as a link
<Button as="a" href="/dashboard">Go to Dashboard</Button>

// Render as a different component
<Button as={Link} to="/settings">Settings</Button>
```

## Common Pitfalls

1. **Forgetting to render `<GlobalStyle />`** — The global styles component must be rendered in your component tree. If you define it but never render it, no global styles are applied.
2. **Importing `css` from the wrong package** — The `css` helper must be imported from `styled-components`, not confused with other CSS-related imports.

## Best Practices

1. **Use `createGlobalStyle` for resets and base styles only** — Keep global styles minimal. Component-specific styles should be in styled components, not in the global stylesheet.
2. **Create a shared mixins file** — Define reusable CSS fragments (`truncate`, `flexCenter`, media queries) in a shared file and import them where needed. This promotes consistency.

## Summary

- `createGlobalStyle` injects global CSS like resets and base typography when rendered as a component.
- `keyframes` creates scoped animation names, and `css` creates reusable style fragments.
- The `as` prop lets any styled component render as a different HTML element or React component while keeping its styles.

## Code Examples

**Keyframes, css helper, and as prop for polymorphism**

```tsx
import styled, { keyframes, css, createGlobalStyle } from "styled-components";

// Reusable animation
const fadeIn = keyframes`
  from { opacity: 0; }
  to { opacity: 1; }
`;

// Reusable CSS fragment
const cardBase = css`
  border-radius: 8px;
  padding: 16px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
`;

// Using both
const Card = styled.div`
  ${cardBase}
  animation: ${fadeIn} 0.3s ease-out;
  background: white;
`;

// Polymorphic rendering with "as"
const Action = styled.button`
  padding: 8px 16px;
  background: #2563eb;
  color: white;
  border: none;
  border-radius: 4px;
`;

// <Action as="a" href="/home">Home</Action> renders as <a>
```


## Resources

- [styled-components API Reference](https://styled-components.com/docs/api) — Full API reference for styled-components

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*