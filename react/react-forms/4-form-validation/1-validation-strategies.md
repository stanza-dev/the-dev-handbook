---
source_course: "react-forms"
source_lesson: "react-forms-validation-strategies"
---

# Validation Strategies

Form validation ensures users submit correct data. There are several strategies for when and how to validate.

## When to Validate

### 1. On Submit
Validate everything when the form is submitted.

```jsx
function handleSubmit(e) {
  e.preventDefault();
  const errors = validate(formData);
  
  if (Object.keys(errors).length > 0) {
    setErrors(errors);
    return;
  }
  
  // Submit form
}
```

**Pros**: Less intrusive, validates everything at once
**Cons**: User discovers all errors at the end

### 2. On Blur (Field Exit)
Validate when the user leaves a field.

```jsx
function handleBlur(e) {
  const { name, value } = e.target;
  const error = validateField(name, value);
  setErrors(prev => ({ ...prev, [name]: error }));
}
```

**Pros**: Immediate feedback after completing a field
**Cons**: Can feel aggressive

### 3. On Change (Real-time)
Validate on every keystroke.

```jsx
function handleChange(e) {
  const { name, value } = e.target;
  setFormData(prev => ({ ...prev, [name]: value }));
  
  // Only show errors if field was already touched
  if (touched[name]) {
    const error = validateField(name, value);
    setErrors(prev => ({ ...prev, [name]: error }));
  }
}
```

**Pros**: Instant feedback
**Cons**: Can be distracting while typing

### 4. Hybrid Approach (Recommended)
Combine strategies for best UX:

```jsx
// Show errors after blur, clear errors on change
function handleBlur(e) {
  setTouched(prev => ({ ...prev, [e.target.name]: true }));
  validateField(e.target.name);
}

function handleChange(e) {
  const { name, value } = e.target;
  setFormData(prev => ({ ...prev, [name]: value }));
  
  // Clear error when user starts fixing
  if (errors[name]) {
    setErrors(prev => ({ ...prev, [name]: null }));
  }
}
```

📚 **Learn more**: [Reacting to Input with State](https://react.dev/learn/reacting-to-input-with-state)

---

> 📘 *This lesson is part of the [React Form Mastery](https://stanza.dev/courses/react-forms) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*