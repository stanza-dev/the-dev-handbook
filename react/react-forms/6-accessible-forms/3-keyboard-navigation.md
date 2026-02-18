---
source_course: "react-forms"
source_lesson: "react-forms-keyboard-navigation"
---

# Keyboard Navigation

Many users navigate forms with keyboards only. Ensure your forms work without a mouse!

## Focus Management

### Auto-focus First Field

```jsx
function Form() {
  const firstInputRef = useRef(null);
  
  useEffect(() => {
    firstInputRef.current?.focus();
  }, []);

  return (
    <form>
      <input ref={firstInputRef} name="name" />
    </form>
  );
}
```

### Focus on Error

When validation fails, focus the first error:

```jsx
function handleSubmit(e) {
  e.preventDefault();
  const errors = validate(formData);
  
  if (Object.keys(errors).length > 0) {
    // Find first field with error and focus it
    const firstErrorField = Object.keys(errors)[0];
    document.getElementById(firstErrorField)?.focus();
  }
}
```

### Focus Visible Styles

```css
/* Never do this! */
*:focus {
  outline: none; /* ❌ Removes focus indicator */
}

/* Do this instead */
input:focus {
  outline: 2px solid #3b82f6;
  outline-offset: 2px;
}

/* Modern approach: only show for keyboard */
input:focus-visible {
  outline: 2px solid #3b82f6;
  outline-offset: 2px;
}
```

## Tab Order

Use natural DOM order. Avoid positive `tabindex` values:

```jsx
// ❌ Bad: Custom tab order
<input tabIndex={3} />
<input tabIndex={1} />
<input tabIndex={2} />

// ✅ Good: Natural order
<input /> {/* tabIndex=0 by default */}
<input />
<input />

// ✅ OK: Remove from tab order when needed
<input tabIndex={-1} /> {/* Skip this input */}
```

## Skip to Error Summary

```jsx
function Form() {
  const errorSummaryRef = useRef(null);
  const [errors, setErrors] = useState({});

  function handleSubmit(e) {
    e.preventDefault();
    const newErrors = validate(formData);
    setErrors(newErrors);
    
    if (Object.keys(newErrors).length > 0) {
      // Focus error summary
      errorSummaryRef.current?.focus();
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      {Object.keys(errors).length > 0 && (
        <div
          ref={errorSummaryRef}
          tabIndex={-1}
          role="alert"
          className="error-summary"
        >
          <h2>Please fix the following errors:</h2>
          <ul>
            {Object.entries(errors).map(([field, error]) => (
              <li key={field}>
                <a href={`#${field}`}>{error}</a>
              </li>
            ))}
          </ul>
        </div>
      )}
      {/* Form fields */}
    </form>
  );
}
```

## Enter to Submit

Forms submit on Enter by default, but ensure you have a submit button:

```jsx
// ✅ Works with Enter key
<form onSubmit={handleSubmit}>
  <input name="search" />
  <button type="submit">Search</button>
</form>
```

---

> 📘 *This lesson is part of the [React Form Mastery](https://stanza.dev/courses/react-forms) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*