---
source_course: "react-intermediate"
source_lesson: "react-tailwind-intro"
---

# Tailwind CSS with React

## Introduction

Tailwind CSS takes a radically different approach to styling: instead of writing custom CSS classes like `.card-header` or `.btn-primary`, you compose styles directly in your markup using small, single-purpose utility classes like `flex`, `p-4`, `bg-white`, and `rounded-lg`. This might look verbose at first, but it eliminates the mental overhead of naming classes, prevents style conflicts, and keeps your styles tightly colocated with your markup. Tailwind has become one of the most popular styling approaches in the React ecosystem.

## Key Concepts

- **Utility-first**: Every class does one thing (`p-4` = padding, `text-lg` = font size, `bg-blue-500` = background color). You combine them to build any design.
- **Responsive prefixes**: Add breakpoint prefixes like `md:`, `lg:`, `xl:` to apply styles at specific screen sizes: `md:grid-cols-2` applies only on medium screens and up.
- **State variants**: Prefix with `hover:`, `focus:`, `active:`, `disabled:` to style interactive states: `hover:bg-blue-700`.
- **PurgeCSS / Content scanning**: Tailwind's build step scans your files and removes unused utility classes, producing tiny production CSS bundles (typically under 10KB).

## Real World Context

A product team building a dashboard might spend hours debating class naming conventions (BEM? OOCSS? Utility-first-ish?), creating a global stylesheet that grows to 500KB, and fighting specificity wars. With Tailwind, there are no naming decisions — you use the framework's predefined utility classes. The design system is built into the framework via its configuration (spacing scale, color palette, typography scale), ensuring consistency across the entire team without manual coordination.

## Deep Dive

Basic component styling with Tailwind:

```tsx
function Card({ title, children }: { title: string; children: React.ReactNode }) {
  return (
    <div className="bg-white rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow">
      <h2 className="text-xl font-bold text-gray-800 mb-4">{title}</h2>
      <div className="text-gray-600 leading-relaxed">{children}</div>
    </div>
  );
}
```

Responsive design uses mobile-first breakpoint prefixes:

```tsx
function ResponsiveGrid({ items }: { items: Item[] }) {
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
      {items.map((item) => (
        <Card key={item.id} title={item.title}>
          {item.description}
        </Card>
      ))}
    </div>
  );
}
```

State variants for interactivity:

```tsx
function Button({ variant = "primary", children, ...props }: ButtonProps) {
  const baseClasses = "px-4 py-2 rounded-md font-medium transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2";

  const variants = {
    primary: "bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500",
    secondary: "bg-gray-200 text-gray-800 hover:bg-gray-300 focus:ring-gray-500",
    danger: "bg-red-600 text-white hover:bg-red-700 focus:ring-red-500",
  };

  return (
    <button className={`${baseClasses} ${variants[variant]}`} {...props}>
      {children}
    </button>
  );
}
```

Dark mode support:

```tsx
function ThemeCard({ children }: { children: React.ReactNode }) {
  return (
    <div className="bg-white dark:bg-gray-800 text-gray-900 dark:text-gray-100 rounded-lg p-6 border border-gray-200 dark:border-gray-700">
      {children}
    </div>
  );
}
```

## Common Pitfalls

1. **String interpolation breaks PurgeCSS** — Writing `bg-${color}-500` prevents Tailwind from detecting the class. Always use complete class names: use a mapping object or conditional expressions with full class strings.
2. **Over-long className strings** — Very complex components can end up with 20+ classes on a single element. Extract reusable patterns into components or use the `@apply` directive in a CSS file for truly reusable base styles.

## Best Practices

1. **Extract components, not CSS** — Instead of creating a `.card` CSS class, create a `<Card>` React component with the Tailwind classes. This keeps styles and behavior together.
2. **Use the design system tokens** — Stick to Tailwind's spacing scale (`p-1` through `p-12`), color palette, and typography. Avoid arbitrary values like `w-[347px]` unless absolutely necessary.

## Summary

- Tailwind CSS uses small, composable utility classes applied directly in markup, eliminating custom CSS class naming.
- Responsive prefixes (`md:`, `lg:`) and state variants (`hover:`, `focus:`) provide a complete styling system without writing any CSS files.
- Tailwind's build tool removes unused classes, producing tiny production bundles despite having thousands of available utilities.

## Code Examples

**Product card component styled entirely with Tailwind utilities**

```tsx
function ProductCard({ product }: { product: Product }) {
  return (
    <div className="bg-white rounded-lg shadow-md overflow-hidden hover:shadow-xl transition-shadow">
      <img
        src={product.image}
        alt={product.name}
        className="w-full h-48 object-cover"
      />
      <div className="p-4">
        <h3 className="font-bold text-lg text-gray-800">{product.name}</h3>
        <p className="text-gray-500 text-sm mt-1">{product.description}</p>
        <div className="mt-4 flex items-center justify-between">
          <span className="text-xl font-bold text-blue-600">
            ${product.price}
          </span>
          <button className="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 transition-colors">
            Add to Cart
          </button>
        </div>
      </div>
    </div>
  );
}
```


## Resources

- [Tailwind CSS Documentation](https://tailwindcss.com/docs) — Official Tailwind CSS documentation
- [Utility-First Fundamentals](https://tailwindcss.com/docs/utility-first) — Tailwind's utility-first methodology explained

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*