---
source_course: "react"
source_lesson: "react-rendering-lists"
---

# Rendering Lists

## Introduction

Most applications need to display collections of data — product listings, chat messages, search results, navigation menus. In React, you render lists by transforming arrays of data into arrays of JSX elements using JavaScript's `map()` method. The key to doing this correctly is understanding React's `key` prop, which is critical for performance and correctness.

## Key Concepts

- **Array.map()**: The primary method for transforming an array of data into an array of JSX elements.
- **Key Prop**: A special string attribute that helps React identify which items have changed, been added, or been removed during reconciliation.
- **Stable Identity**: Keys should come from your data (database IDs, unique slugs) rather than from the rendering process (array indices, random values).
- **Filtering Before Mapping**: Use `Array.filter()` before `map()` to conditionally render subsets of a list.

## Real World Context

An e-commerce search results page might display 50 products. When the user applies a filter, some products are removed and the remaining ones re-order. React uses keys to match each product element in the old list to its counterpart in the new list. Without proper keys, React might reuse DOM nodes incorrectly, causing visual glitches like the wrong image appearing next to a product name, or input fields retaining stale values.

## Deep Dive

### Basic List Rendering

```tsx
type Todo = { id: number; text: string; completed: boolean };

function TodoList({ todos }: { todos: Todo[] }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          {todo.completed ? '\u2705' : '\u2B1C'} {todo.text}
        </li>
      ))}
    </ul>
  );
}
```

Each `<li>` receives a `key` equal to the todo's unique `id`. This lets React efficiently update, reorder, or remove individual items.

### Filtering and Mapping

You can chain `filter()` and `map()` to render a subset:

```tsx
function ActiveTodos({ todos }: { todos: Todo[] }) {
  return (
    <ul>
      {todos
        .filter(todo => !todo.completed)
        .map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
    </ul>
  );
}
```

### Why Index Keys Are Dangerous

Using the array index as a key works only when the list is static and never reordered. If items can be added, removed, or sorted, index keys cause React to mismatch elements:

```tsx
// Dangerous when items can reorder
{items.map((item, index) => (
  <input key={index} defaultValue={item.name} />
))}

// Safe: use a stable identifier
{items.map(item => (
  <input key={item.id} defaultValue={item.name} />
))}
```

When you use index keys and an item is inserted at the beginning, every subsequent element gets a new index, causing React to re-render all of them and potentially lose uncontrolled input state.

### Rendering Nested Lists

For nested data, use unique keys at each level:

```tsx
function CourseList({ courses }) {
  return courses.map(course => (
    <section key={course.id}>
      <h2>{course.title}</h2>
      <ul>
        {course.lessons.map(lesson => (
          <li key={lesson.id}>{lesson.title}</li>
        ))}
      </ul>
    </section>
  ));
}
```

## Common Pitfalls

1. **Using array index as key for dynamic lists** — This leads to incorrect DOM reuse when items are reordered, inserted, or deleted. Always use a stable unique identifier from your data.
2. **Forgetting the key prop entirely** — React will warn you in the console, but more importantly, without keys React cannot efficiently reconcile list changes and may cause subtle bugs.

## Best Practices

1. **Use database IDs or unique slugs as keys** — These are stable across re-renders and uniquely identify each item. If your data does not have IDs, generate them when the data is created, not during rendering.
2. **Keep the mapped JSX simple** — If the body of your `map()` callback is more than a few lines, extract it into a separate component for readability and testability.

## Summary

- Use `Array.map()` to transform data arrays into JSX element arrays.
- Every list item must have a unique, stable `key` prop derived from the data itself (not the array index).
- Chain `filter()` before `map()` to conditionally render subsets of a list.

## Code Examples

**Filtering and rendering a product list with stable keys**

```tsx
type Product = { id: string; name: string; price: number; inStock: boolean };

function ProductGrid({ products }: { products: Product[] }) {
  const available = products.filter(p => p.inStock);

  return (
    <div className="grid">
      {available.map(product => (
        <div key={product.id} className="product-card">
          <h3>{product.name}</h3>
          <p>${product.price.toFixed(2)}</p>
        </div>
      ))}
    </div>
  );
}
```


## Resources

- [Rendering Lists](https://react.dev/learn/rendering-lists) — Official guide on rendering lists and using keys

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*