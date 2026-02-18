---
source_course: "react"
source_lesson: "react-tailwind-intro"
---

# Tailwind CSS

A utility-first CSS framework for rapidly building custom designs.

## Philosophy

Instead of writing custom CSS, you compose styles using utility classes:

```jsx
<div className="flex items-center justify-between p-4 bg-white rounded-lg shadow">
```

## Benefits

- **Rapid development**: No context switching to CSS files
- **Consistency**: Design tokens built-in
- **Small bundle**: PurgeCSS removes unused styles
- **Responsive**: Mobile-first breakpoint system

## Code Examples

**Tailwind CSS patterns in React**

```tsx
// Card component with Tailwind
function Card({ title, children }) {
  return (
    <div className="bg-white rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow">
      <h2 className="text-xl font-bold text-gray-800 mb-4">
        {title}
      </h2>
      <div className="text-gray-600">
        {children}
      </div>
    </div>
  );
}

// Button with variants using Tailwind
function Button({ variant = 'primary', children, ...props }) {
  const baseClasses = 'px-4 py-2 rounded-md font-medium transition-colors';
  
  const variants = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700',
    secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300',
    danger: 'bg-red-600 text-white hover:bg-red-700',
  };
  
  return (
    <button 
      className={`${baseClasses} ${variants[variant]}`}
      {...props}
    >
      {children}
    </button>
  );
}

// Responsive design
function ResponsiveGrid({ items }) {
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      {items.map(item => (
        <Card key={item.id} title={item.title}>
          {item.description}
        </Card>
      ))}
    </div>
  );
}
```


## Resources

- [Tailwind CSS Documentation](https://tailwindcss.com/docs) — Official Tailwind docs

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*