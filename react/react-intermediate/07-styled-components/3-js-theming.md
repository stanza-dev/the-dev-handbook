---
source_course: "react-intermediate"
source_lesson: "react-css-in-js-theming"
---

# CSS-in-JS Theming

## Introduction

One of the strongest features of CSS-in-JS libraries is their built-in theming support. Styled Components provides a `ThemeProvider` that makes a theme object available to every styled component in its subtree via React context. This enables consistent design tokens (colors, spacing, typography) across your entire application and makes implementing features like dark mode straightforward.

## Key Concepts

- **`ThemeProvider`**: A context provider from styled-components that makes a theme object accessible to all styled components in its children.
- **Theme object**: A plain JavaScript object containing design tokens — colors, spacing, font sizes, border radii, shadows, and any other design values.
- **`props.theme`**: Inside a styled component's template literal, you access the theme via `${props => props.theme.colors.primary}`.
- **Theme switching**: Changing the theme object passed to `ThemeProvider` instantly updates all styled components that reference theme values.

## Real World Context

A SaaS product needs both light and dark themes, plus a high-contrast accessibility mode, and each customer can customize their brand color. Instead of maintaining three separate CSS files or hundreds of CSS variables, you define three theme objects with the same shape but different values, and let `ThemeProvider` handle the rest. Styled components reference `theme.colors.primary` and automatically get the right value for the active theme.

## Deep Dive

Define your theme:

```tsx
const lightTheme = {
  colors: {
    background: "#ffffff",
    surface: "#f9fafb",
    text: "#111827",
    textSecondary: "#6b7280",
    primary: "#2563eb",
    primaryHover: "#1d4ed8",
    danger: "#dc2626",
    border: "#e5e7eb",
  },
  spacing: {
    xs: "4px",
    sm: "8px",
    md: "16px",
    lg: "24px",
    xl: "32px",
  },
  borderRadius: {
    sm: "4px",
    md: "8px",
    lg: "16px",
    full: "9999px",
  },
};

const darkTheme = {
  ...lightTheme,
  colors: {
    background: "#111827",
    surface: "#1f2937",
    text: "#f9fafb",
    textSecondary: "#9ca3af",
    primary: "#3b82f6",
    primaryHover: "#60a5fa",
    danger: "#ef4444",
    border: "#374151",
  },
};
```

Set up the provider with theme switching:

```tsx
import { ThemeProvider } from "styled-components";

function App() {
  const [isDark, setIsDark] = useState(false);
  const theme = isDark ? darkTheme : lightTheme;

  return (
    <ThemeProvider theme={theme}>
      <GlobalStyle />
      <Header onToggleTheme={() => setIsDark(!isDark)} />
      <MainContent />
    </ThemeProvider>
  );
}
```

Use theme values in styled components:

```tsx
const Card = styled.div`
  background: ${(p) => p.theme.colors.surface};
  color: ${(p) => p.theme.colors.text};
  border: 1px solid ${(p) => p.theme.colors.border};
  border-radius: ${(p) => p.theme.borderRadius.md};
  padding: ${(p) => p.theme.spacing.md};
`;

const Button = styled.button`
  background: ${(p) => p.theme.colors.primary};
  color: white;
  padding: ${(p) => p.theme.spacing.sm} ${(p) => p.theme.spacing.md};
  border: none;
  border-radius: ${(p) => p.theme.borderRadius.sm};
  &:hover {
    background: ${(p) => p.theme.colors.primaryHover};
  }
`;
```

Access the theme in regular components with `useTheme`:

```tsx
import { useTheme } from "styled-components";

function StatusIndicator({ active }: { active: boolean }) {
  const theme = useTheme();
  return (
    <span style={{ color: active ? theme.colors.primary : theme.colors.textSecondary }}>
      {active ? "Active" : "Inactive"}
    </span>
  );
}
```

## Common Pitfalls

1. **Inconsistent theme object shapes** — If your light theme has `colors.primary` but your dark theme uses `colors.main`, switching themes will break components. Always use the same object shape for all themes.
2. **Accessing theme without ThemeProvider** — Styled components that reference `props.theme` will get `undefined` if there is no `ThemeProvider` ancestor. Ensure the provider wraps your entire app.

## Best Practices

1. **Type your theme with TypeScript** — Define an interface for your theme and extend styled-components' `DefaultTheme` declaration so `props.theme` is fully typed with autocomplete.
2. **Use semantic token names** — Name theme values by purpose (`colors.surface`, `colors.textSecondary`) rather than appearance (`colors.white`, `colors.gray500`). This makes themes interchangeable.

## Summary

- `ThemeProvider` makes a theme object available to all styled components via React context.
- Theme objects should have consistent shapes across variants (light, dark) with semantic token names.
- Theme switching is as simple as changing the object passed to `ThemeProvider`, instantly updating all styled components.

## Code Examples

**Theme switching with ThemeProvider and styled components**

```tsx
import styled, { ThemeProvider } from "styled-components";

const light = {
  bg: "#ffffff", text: "#111827", primary: "#2563eb", border: "#e5e7eb"
};
const dark = {
  bg: "#111827", text: "#f9fafb", primary: "#3b82f6", border: "#374151"
};

const Page = styled.div`
  background: ${p => p.theme.bg};
  color: ${p => p.theme.text};
  min-height: 100vh;
  padding: 24px;
`;

const Card = styled.div`
  background: ${p => p.theme.bg};
  border: 1px solid ${p => p.theme.border};
  border-radius: 8px;
  padding: 16px;
`;

function App() {
  const [isDark, setIsDark] = useState(false);
  return (
    <ThemeProvider theme={isDark ? dark : light}>
      <Page>
        <button onClick={() => setIsDark(!isDark)}>Toggle Theme</button>
        <Card>Themed content</Card>
      </Page>
    </ThemeProvider>
  );
}
```


## Resources

- [Theming](https://styled-components.com/docs/advanced#theming) — styled-components theming documentation

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*