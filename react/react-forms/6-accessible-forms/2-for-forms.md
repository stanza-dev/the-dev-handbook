---
source_course: "react-forms"
source_lesson: "react-forms-aria-for-forms"
---

# ARIA for Forms

ARIA (Accessible Rich Internet Applications) attributes help screen readers understand dynamic form states.

## Key ARIA Attributes for Forms

### aria-invalid

Indicates the input has a validation error:

```jsx
<input
  aria-invalid={hasError}
  aria-describedby={hasError ? 'email-error' : undefined}
/>
{hasError && (
  <span id="email-error" role="alert">
    Please enter a valid email
  </span>
)}
```

### aria-describedby

Links input to helpful text:

```jsx
<label htmlFor="password">Password</label>
<input
  id="password"
  type="password"
  aria-describedby="password-hint password-error"
/>
<span id="password-hint">
  Must be at least 8 characters
</span>
{error && (
  <span id="password-error" role="alert">
    {error}
  </span>
)}
```

### aria-required

```jsx
<input
  required
  aria-required="true"
/>
```

### role="alert"

Announces content immediately to screen readers:

```jsx
{error && (
  <div role="alert" className="error">
    {error}
  </div>
)}
```

### aria-live

For dynamic updates that aren't alerts:

```jsx
<div aria-live="polite">
  {characterCount}/500 characters
</div>
```

## Complete Accessible Input Component

```jsx
function AccessibleInput({
  id,
  label,
  error,
  hint,
  required,
  ...props
}) {
  const errorId = `${id}-error`;
  const hintId = `${id}-hint`;
  
  const describedBy = [
    hint && hintId,
    error && errorId
  ].filter(Boolean).join(' ') || undefined;

  return (
    <div className="form-field">
      <label htmlFor={id}>
        {label}
        {required && (
          <span aria-hidden="true"> *</span>
        )}
      </label>
      
      <input
        id={id}
        required={required}
        aria-required={required}
        aria-invalid={!!error}
        aria-describedby={describedBy}
        {...props}
      />
      
      {hint && (
        <span id={hintId} className="hint">
          {hint}
        </span>
      )}
      
      {error && (
        <span id={errorId} className="error" role="alert">
          {error}
        </span>
      )}
    </div>
  );
}
```

---

> 📘 *This lesson is part of the [React Form Mastery](https://stanza.dev/courses/react-forms) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*