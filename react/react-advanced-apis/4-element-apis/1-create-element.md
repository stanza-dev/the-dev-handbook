---
source_course: "react-advanced-apis"
source_lesson: "react-advanced-apis-create-element"
---

# createElement: How JSX Really Works

`createElement` creates a React element. JSX compiles to `createElement` calls, so understanding it helps you understand React deeply.

## JSX Compilation

```tsx
// This JSX:
<Button color="blue" size="large">
  Click me
</Button>

// Compiles to:
createElement(
  Button,
  { color: 'blue', size: 'large' },
  'Click me'
);
```

## Function Signature

```tsx
createElement(type, props, ...children)
```

- **type**: Component, tag name ('div'), or Fragment
- **props**: Object of props (or null)
- **children**: Child elements (rest parameters)

## When to Use createElement

Most of the time, use JSX. But `createElement` is useful for:

### 1. Dynamic Component Selection

```tsx
function DynamicComponent({ componentType, ...props }) {
  // Can't do <componentType /> in JSX
  return createElement(componentType, props);
}

// Usage
<DynamicComponent componentType={Button} color="blue" />
```

### 2. Programmatic Element Creation

```tsx
function createFormFields(schema) {
  return schema.map(field =>
    createElement(
      field.type === 'textarea' ? 'textarea' : 'input',
      {
        key: field.name,
        name: field.name,
        type: field.inputType,
        placeholder: field.placeholder
      }
    )
  );
}
```

### 3. Higher-Order Components

```tsx
function withLogging(WrappedComponent) {
  return function LoggingComponent(props) {
    useEffect(() => {
      console.log('Rendered:', WrappedComponent.name);
    });
    
    return createElement(WrappedComponent, props);
  };
}
```

## createElement vs JSX

| Aspect | JSX | createElement |
|--------|-----|---------------|
| Readability | ✅ More readable | ❌ Verbose |
| Dynamic types | ❌ Needs workarounds | ✅ Natural |
| Build step | ✅ Required | ❌ Not required |
| Performance | Same | Same |

## The Element Object

`createElement` returns a plain object:

```tsx
const element = createElement('div', { className: 'box' }, 'Hello');

console.log(element);
// {
//   type: 'div',
//   props: { className: 'box', children: 'Hello' },
//   key: null,
//   ref: null,
//   ...
// }
```

Elements are immutable descriptions of UI, not actual DOM nodes.

## Resources

- [createElement API Reference](https://react.dev/reference/react/createElement) — Official React documentation for createElement

---

> 📘 *This lesson is part of the [React Advanced APIs](https://stanza.dev/courses/react-advanced-apis) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*