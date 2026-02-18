---
source_course: "react-advanced-apis"
source_lesson: "react-advanced-apis-create-root"
---

# createRoot: Bootstrapping React Applications

`createRoot` is the primary API for rendering React applications in the browser. It creates a root that manages a React tree inside a DOM node.

## Basic Usage

```tsx
import { createRoot } from 'react-dom/client';
import App from './App';

const domNode = document.getElementById('root');
const root = createRoot(domNode);
root.render(<App />);
```

## The Root Object

The `createRoot` function returns a root object with two methods:

### root.render(reactNode)

Renders a React element into the root. Call it to display your app:

```tsx
root.render(<App />);
```

You can call `render` multiple times to update the UI:

```tsx
// Initial render
root.render(<App page="home" />);

// Later, update with new props
root.render(<App page="about" />);
```

### root.unmount()

Destroys the React tree and cleans up all resources:

```tsx
root.unmount();
```

This runs all cleanup functions from effects and removes the React tree from the DOM.

## Configuration Options

`createRoot` accepts an optional second argument for configuration:

```tsx
const root = createRoot(domNode, {
  // Called when React catches an error in an Error Boundary
  onCaughtError: (error, errorInfo) => {
    console.error('Caught error:', error);
    logErrorToService(error, errorInfo.componentStack);
  },
  
  // Called for uncaught errors (crashes the app)
  onUncaughtError: (error, errorInfo) => {
    console.error('Uncaught error:', error);
    showErrorDialog(error);
  },
  
  // Called when React recovers from errors automatically
  onRecoverableError: (error, errorInfo) => {
    console.warn('Recoverable error:', error);
  },
  
  // Prefix for useId() generated IDs
  identifierPrefix: 'my-app-'
});
```

## Important Considerations

1. **Single Root Per Container**: Each DOM container should have only one React root
2. **Full Control**: React takes full control of the DOM inside the root
3. **Don't Mix**: Avoid mixing React rendering with direct DOM manipulation inside the root

```tsx
// ✅ Correct - React manages everything inside #root
<div id="root"></div>

// 🔴 Avoid - Don't manually modify DOM inside the root
root.render(<App />);
document.getElementById('root').innerHTML += '<div>Extra</div>'; // Bad!
```

## Resources

- [createRoot API Reference](https://react.dev/reference/react-dom/client/createRoot) — Official React documentation for createRoot

---

> 📘 *This lesson is part of the [React Advanced APIs](https://stanza.dev/courses/react-advanced-apis) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*