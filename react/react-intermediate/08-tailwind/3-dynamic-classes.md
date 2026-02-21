---
source_course: "react-intermediate"
source_lesson: "react-tailwind-dynamic-classes"
---

# Dynamic Classes in Tailwind

## Introduction

One of the trickiest aspects of using Tailwind with React is handling dynamic class names. Because Tailwind's build tool scans your source files for complete class names to include in the production bundle, you cannot construct class names dynamically with string interpolation. This lesson explains why, and covers the patterns that let you use dynamic styles effectively without breaking Tailwind's purge process.

## Key Concepts

- **Static extraction**: Tailwind's compiler scans your source code as plain text, looking for complete class name strings. It does not execute your JavaScript, so it cannot resolve template literals or variables.
- **Complete class names**: Always write the full class name (`bg-red-500`) rather than constructing it (`bg-${color}-500`). The compiler must be able to find the exact string in your source.
- **Safelist**: For truly dynamic classes that cannot be written statically, Tailwind's config allows a `safelist` option that forces specific classes to always be included.
- **Arbitrary values**: When Tailwind's design tokens do not cover your need, use bracket notation (`w-[200px]`, `bg-[#1da1f2]`) for one-off values.

## Real World Context

Imagine a chart component where the bar color comes from an API response. The color could be any of 10 predefined values. You cannot write `bg-${apiColor}-500` because Tailwind will not find that class during its scan. Instead, you create a mapping object with all possible colors written as complete strings, and the compiler finds them all.

## Deep Dive

**The problem with dynamic class construction:**

```tsx
// BAD: Tailwind cannot detect these classes
function Badge({ color }: { color: string }) {
  return <span className={`bg-${color}-100 text-${color}-800`}>Badge</span>;
}
// "bg-red-100" never appears as a complete string, so it gets purged!
```

**The solution: complete class name mapping:**

```tsx
// GOOD: All class names are complete strings that Tailwind can find
const colorMap: Record<string, string> = {
  red: "bg-red-100 text-red-800",
  blue: "bg-blue-100 text-blue-800",
  green: "bg-green-100 text-green-800",
  yellow: "bg-yellow-100 text-yellow-800",
  purple: "bg-purple-100 text-purple-800",
};

function Badge({ color }: { color: keyof typeof colorMap }) {
  return <span className={`rounded-full px-2 py-1 text-sm ${colorMap[color]}`}>Badge</span>;
}
```

**Conditional classes (perfectly safe):**

```tsx
// GOOD: Both complete class names are visible in the source
function Alert({ isError }: { isError: boolean }) {
  return (
    <div className={isError ? "bg-red-100 text-red-800" : "bg-blue-100 text-blue-800"}>
      {isError ? "Error!" : "Info"}
    </div>
  );
}
```

**Arbitrary values for one-off needs:**

```tsx
function ProgressBar({ percent }: { percent: number }) {
  return (
    <div className="w-full bg-gray-200 rounded-full h-2">
      <div
        className="bg-blue-600 rounded-full h-2 transition-all"
        style={{ width: `${percent}%` }}
      />
    </div>
  );
}
```

**Safelist for truly dynamic cases:**

```tsx
// tailwind.config.ts
export default {
  safelist: [
    { pattern: /^bg-(red|blue|green|yellow|purple)-(100|500)$/ },
    { pattern: /^text-(red|blue|green|yellow|purple)-800$/ },
  ],
};
```

**When to use inline `style` instead:**

```tsx
// For truly dynamic values (user-chosen colors, calculated positions),
// use the style prop instead of trying to use Tailwind
function ColorPicker({ selectedColor }: { selectedColor: string }) {
  return (
    <div
      className="w-10 h-10 rounded-full border-2 border-white shadow"
      style={{ backgroundColor: selectedColor }}
    />
  );
}
```

## Common Pitfalls

1. **Constructing partial class names** — `text-${size}` or `p-${spacing}` will be purged. Always use complete class strings visible in your source code.
2. **Over-using the safelist** — Safelisting hundreds of classes defeats Tailwind's tree-shaking benefit. Prefer mapping objects with complete class names for the vast majority of cases.

## Best Practices

1. **Use mapping objects for dynamic values** — Create a const object that maps each possible value to its complete Tailwind class string. This is both PurgeCSS-safe and type-safe with TypeScript.
2. **Use `style` for truly variable values** — User-selected colors, calculated positions, and API-driven dimensions should use the inline `style` prop, not Tailwind classes. Tailwind is for design system tokens, not arbitrary runtime values.

## Summary

- Tailwind scans source code as text, so class names must appear as complete strings — never construct them with string interpolation.
- Use mapping objects with complete class names for dynamic variants, and conditional expressions for toggling between known states.
- For truly variable values (user input, API data), use the inline `style` prop instead of trying to force Tailwind classes.

## Code Examples

**Safe dynamic class patterns and when to use style prop**

```tsx
// Mapping object — safe for Tailwind's compiler
const statusColors = {
  active: "bg-green-100 text-green-800 border-green-200",
  inactive: "bg-gray-100 text-gray-600 border-gray-200",
  pending: "bg-yellow-100 text-yellow-800 border-yellow-200",
  error: "bg-red-100 text-red-800 border-red-200",
} as const;

type Status = keyof typeof statusColors;

function StatusBadge({ status }: { status: Status }) {
  return (
    <span className={`inline-flex items-center rounded-full px-3 py-1 text-sm font-medium border ${statusColors[status]}`}>
      {status}
    </span>
  );
}

// For truly dynamic values, use style prop
function Avatar({ color }: { color: string }) {
  return (
    <div
      className="w-10 h-10 rounded-full flex items-center justify-center text-white font-bold"
      style={{ backgroundColor: color }}
    >
      A
    </div>
  );
}
```


## Resources

- [Dynamic Class Names](https://tailwindcss.com/docs/detecting-classes-in-source-files) — Tailwind docs on how class detection works

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*