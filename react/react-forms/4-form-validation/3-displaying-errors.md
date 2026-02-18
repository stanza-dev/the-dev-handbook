---
source_course: "react-forms"
source_lesson: "react-forms-displaying-errors"
---

# Displaying Validation Errors

How you display errors significantly impacts user experience. Let's explore best practices.

## Inline Error Messages

```jsx
function FormField({ name, label, error, touched, ...props }) {
  const showError = touched && error;
  
  return (
    <div className="form-field">
      <label htmlFor={name}>{label}</label>
      <input
        id={name}
        name={name}
        aria-invalid={showError}
        aria-describedby={showError ? `${name}-error` : undefined}
        className={showError ? 'input-error' : ''}
        {...props}
      />
      {showError && (
        <span 
          id={`${name}-error`} 
          className="error-message"
          role="alert"
        >
          {error}
        </span>
      )}
    </div>
  );
}
```

## Error Summary

Show all errors at the top of the form:

```jsx
function ErrorSummary({ errors }) {
  const errorList = Object.entries(errors)
    .filter(([_, error]) => error);
  
  if (errorList.length === 0) return null;
  
  return (
    <div className="error-summary" role="alert">
      <h4>Please fix the following errors:</h4>
      <ul>
        {errorList.map(([field, error]) => (
          <li key={field}>
            <a href={`#${field}`}>{error}</a>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## Visual Indicators

```css
/* Error state */
.input-error {
  border-color: #dc2626;
  background-color: #fef2f2;
}

.error-message {
  color: #dc2626;
  font-size: 0.875rem;
  margin-top: 0.25rem;
}

/* Success state */
.input-valid {
  border-color: #16a34a;
}

/* Focus + error */
.input-error:focus {
  outline-color: #dc2626;
  box-shadow: 0 0 0 3px rgba(220, 38, 38, 0.2);
}
```

## Real-time Password Strength

```jsx
function PasswordStrength({ password }) {
  const checks = [
    { test: /.{8,}/, label: 'At least 8 characters' },
    { test: /[A-Z]/, label: 'One uppercase letter' },
    { test: /[a-z]/, label: 'One lowercase letter' },
    { test: /[0-9]/, label: 'One number' },
    { test: /[^A-Za-z0-9]/, label: 'One special character' }
  ];

  return (
    <ul className="password-checklist">
      {checks.map(({ test, label }) => (
        <li 
          key={label}
          className={test.test(password) ? 'valid' : 'invalid'}
        >
          {test.test(password) ? '✓' : '○'} {label}
        </li>
      ))}
    </ul>
  );
}
```

## Accessibility Considerations

1. Use `aria-invalid="true"` on invalid inputs
2. Connect errors with `aria-describedby`
3. Use `role="alert"` for dynamic error messages
4. Ensure sufficient color contrast
5. Don't rely solely on color to indicate errors

---

> 📘 *This lesson is part of the [React Form Mastery](https://stanza.dev/courses/react-forms) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*