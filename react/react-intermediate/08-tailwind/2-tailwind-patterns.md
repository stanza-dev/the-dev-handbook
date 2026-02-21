---
source_course: "react-intermediate"
source_lesson: "react-tailwind-patterns"
---

# Tailwind Patterns in React

## Introduction

Using Tailwind CSS effectively in React requires more than just knowing the utility classes. You need patterns for managing className complexity, creating variant systems for reusable components, and organizing your Tailwind configuration for consistency. This lesson covers the patterns that production React applications use to get the most out of Tailwind without drowning in className strings.

## Key Concepts

- **Variant objects**: Creating JavaScript objects that map variant names to Tailwind class strings for type-safe, organized styling.
- **`cn` / `clsx` utility**: A helper function for conditionally combining class names, essential for Tailwind in React.
- **`tailwind-merge`**: A library that intelligently merges Tailwind classes, resolving conflicts (e.g., `p-4` and `p-2` resolves to `p-2`).
- **Component extraction**: The primary pattern for reuse in Tailwind — extract a React component rather than a CSS class.

## Real World Context

A design system built with Tailwind needs a Button component with 5 variants, 3 sizes, loading state, and optional full-width. Without a pattern, you end up with an unreadable mess of ternary operators and string concatenation. Variant objects and a merge utility transform this into clean, maintainable code that is easy to extend with new variants.

## Deep Dive

**The `cn` utility (clsx + tailwind-merge):**

```tsx
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

**Variant-based component pattern:**

```tsx
import { cn } from "@/lib/utils";

const buttonVariants = {
  variant: {
    primary: "bg-blue-600 text-white hover:bg-blue-700",
    secondary: "bg-gray-100 text-gray-900 hover:bg-gray-200",
    ghost: "text-gray-600 hover:bg-gray-100 hover:text-gray-900",
    danger: "bg-red-600 text-white hover:bg-red-700",
    outline: "border border-gray-300 text-gray-700 hover:bg-gray-50",
  },
  size: {
    sm: "h-8 px-3 text-sm",
    md: "h-10 px-4 text-sm",
    lg: "h-12 px-6 text-base",
  },
};

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: keyof typeof buttonVariants.variant;
  size?: keyof typeof buttonVariants.size;
  loading?: boolean;
  fullWidth?: boolean;
}

function Button({
  variant = "primary",
  size = "md",
  loading,
  fullWidth,
  className,
  children,
  ...props
}: ButtonProps) {
  return (
    <button
      className={cn(
        "inline-flex items-center justify-center rounded-md font-medium transition-colors",
        "focus:outline-none focus:ring-2 focus:ring-offset-2",
        "disabled:opacity-50 disabled:cursor-not-allowed",
        buttonVariants.variant[variant],
        buttonVariants.size[size],
        fullWidth && "w-full",
        className
      )}
      disabled={loading || props.disabled}
      {...props}
    >
      {loading ? "Loading..." : children}
    </button>
  );
}
```

**Accepting a `className` prop for customization:**

```tsx
// Component authors should always accept className for overrides
function Card({ className, children }: { className?: string; children: React.ReactNode }) {
  return (
    <div className={cn("bg-white rounded-lg shadow p-6", className)}>
      {children}
    </div>
  );
}

// Consumers can override or extend
<Card className="border-2 border-blue-500 p-8">Custom card</Card>
// tailwind-merge resolves p-6 vs p-8 to p-8
```

**Responsive component patterns:**

```tsx
function DashboardLayout({ sidebar, main }: { sidebar: React.ReactNode; main: React.ReactNode }) {
  return (
    <div className="flex flex-col md:flex-row min-h-screen">
      <aside className="w-full md:w-64 lg:w-80 bg-gray-50 border-b md:border-b-0 md:border-r p-4">
        {sidebar}
      </aside>
      <main className="flex-1 p-4 md:p-6 lg:p-8">
        {main}
      </main>
    </div>
  );
}
```

## Common Pitfalls

1. **Not using tailwind-merge** — Without it, conflicting classes like `p-4` (from the component) and `p-8` (from className override) both apply, with unpredictable results. `tailwind-merge` resolves the conflict intelligently.
2. **Hardcoding colors instead of using semantic names** — Using `bg-blue-600` everywhere makes theme changes painful. Define semantic colors in your Tailwind config (`bg-primary`) that map to actual color values.

## Best Practices

1. **Always accept a `className` prop** — Let consumers of your components override or extend styles. Use `cn(defaultClasses, className)` to merge properly.
2. **Use variant objects for multi-variant components** — Map variant names to class strings in a typed object. This is cleaner than nested ternaries and provides TypeScript autocomplete.

## Summary

- The `cn` utility (combining `clsx` and `tailwind-merge`) is essential for conditional and conflict-free class name composition.
- Variant objects map prop values to Tailwind class strings, creating type-safe, organized component APIs.
- Always accept a `className` prop on reusable components to allow consumer customization.

## Code Examples

**Badge component with variant pattern and className override**

```tsx
import { cn } from "@/lib/utils";

// Variant pattern for a Badge component
const badgeVariants = {
  info: "bg-blue-100 text-blue-800",
  success: "bg-green-100 text-green-800",
  warning: "bg-yellow-100 text-yellow-800",
  error: "bg-red-100 text-red-800",
};

function Badge({ variant = "info", className, children }: {
  variant?: keyof typeof badgeVariants;
  className?: string;
  children: React.ReactNode;
}) {
  return (
    <span className={cn(
      "inline-flex items-center rounded-full px-2.5 py-0.5 text-xs font-medium",
      badgeVariants[variant],
      className
    )}>
      {children}
    </span>
  );
}

// Usage
<Badge variant="success">Active</Badge>
<Badge variant="error" className="text-sm">Critical</Badge>
```


## Resources

- [Reusing Styles](https://tailwindcss.com/docs/reusing-styles) — Tailwind docs on extracting reusable style patterns

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*