---
source_course: "react-accessibility"
source_lesson: "react-accessibility-forms-accessibility"
---

# Accessible Forms

Forms are critical for accessibility - users must be able to input data.

## Labels Are Required

```jsx
// ✅ Explicit label (preferred)
<label htmlFor="email">Email address</label>
<input id="email" type="email" />

// ✅ Implicit label (wrapping)
<label>
  Email address
  <input type="email" />
</label>

// ❌ No label - screen readers don't know what this is
<input type="email" placeholder="Email" />
```

## Placeholders Are Not Labels

```jsx
// ❌ Bad: Placeholder as only label
<input placeholder="Email" />

// ✅ Good: Real label with helper placeholder
<label htmlFor="email">Email</label>
<input 
  id="email" 
  placeholder="you@example.com"
/>
```

Why? Placeholders:
- Disappear when typing
- Have poor color contrast
- Can't be announced consistently

## Required Fields

```jsx
<label htmlFor="name">
  Name <span aria-hidden="true">*</span>
</label>
<input 
  id="name" 
  required 
  aria-required="true"
/>
<p id="required-note" className="text-sm">
  * Required fields
</p>
```

## Error Messages

```jsx
function EmailField({ error }) {
  const inputId = 'email';
  const errorId = 'email-error';
  
  return (
    <div>
      <label htmlFor={inputId}>Email</label>
      <input
        id={inputId}
        type="email"
        aria-invalid={!!error}
        aria-describedby={error ? errorId : undefined}
      />
      {error && (
        <p id={errorId} role="alert" className="error">
          {error}
        </p>
      )}
    </div>
  );
}
```

## Fieldsets for Groups

```jsx
<fieldset>
  <legend>Shipping Address</legend>
  
  <label htmlFor="street">Street</label>
  <input id="street" />
  
  <label htmlFor="city">City</label>
  <input id="city" />
</fieldset>

<fieldset>
  <legend>Payment Method</legend>
  
  <label>
    <input type="radio" name="payment" value="card" />
    Credit Card
  </label>
  <label>
    <input type="radio" name="payment" value="paypal" />
    PayPal
  </label>
</fieldset>
```

## Help Text

```jsx
<label htmlFor="password">Password</label>
<input
  id="password"
  type="password"
  aria-describedby="password-help"
/>
<p id="password-help" className="help-text">
  Must be at least 8 characters with one number
</p>
```

---

> 📘 *This lesson is part of the [React Accessibility (a11y)](https://stanza.dev/courses/react-accessibility) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*