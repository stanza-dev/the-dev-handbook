---
source_course: "react-accessibility"
source_lesson: "react-accessibility-live-regions"
---

# ARIA Live Regions

Announce dynamic content changes to screen readers.

## The Problem

Screen readers don't automatically announce:
- Toast notifications
- Loading states
- Form validation errors
- Chat messages
- Real-time updates

## aria-live

```jsx
// Polite: Waits for pause in speech
<div aria-live="polite">
  {statusMessage}
</div>

// Assertive: Interrupts immediately
<div aria-live="assertive">
  {urgentError}
</div>
```

## role="alert"

Shorthand for assertive announcement:

```jsx
// These are equivalent
<div role="alert">Error!</div>
<div aria-live="assertive" aria-atomic="true">Error!</div>
```

## role="status"

Shorthand for polite announcement:

```jsx
// These are equivalent
<div role="status">Saved successfully</div>
<div aria-live="polite" aria-atomic="true">Saved successfully</div>
```

## Live Region Attributes

```jsx
<div
  aria-live="polite"
  aria-atomic="true"    // Announce entire region
  aria-relevant="additions text"  // What changes to announce
>
  {content}
</div>
```

## Toast Notification Example

```jsx
function Toast({ message, type }) {
  if (!message) return null;
  
  return (
    <div
      role={type === 'error' ? 'alert' : 'status'}
      className={`toast toast-${type}`}
    >
      {message}
    </div>
  );
}

// Usage
function App() {
  const [toast, setToast] = useState(null);
  
  const showSuccess = () => {
    setToast({ message: 'Saved!', type: 'success' });
  };
  
  return (
    <div>
      <button onClick={showSuccess}>Save</button>
      <Toast {...toast} />
    </div>
  );
}
```

## Loading State

```jsx
function LoadingStatus({ isLoading }) {
  return (
    <div role="status" aria-live="polite">
      {isLoading ? (
        <>
          <Spinner aria-hidden="true" />
          <span>Loading...</span>
        </>
      ) : (
        <span className="sr-only">Content loaded</span>
      )}
    </div>
  );
}
```

## Form Validation

```jsx
function FormField({ error }) {
  return (
    <div>
      <input aria-invalid={!!error} />
      {/* Live region for errors */}
      <div aria-live="polite" aria-atomic="true">
        {error && (
          <span className="error">{error}</span>
        )}
      </div>
    </div>
  );
}
```

---

> 📘 *This lesson is part of the [React Accessibility (a11y)](https://stanza.dev/courses/react-accessibility) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*