---
source_course: "react-forms"
source_lesson: "react-forms-form-labels-and-structure"
---

# Labels & Structure

Proper labels and structure make forms usable for everyone, including users of screen readers and keyboard-only users.

## Always Use Labels

Every input needs an associated label:

```jsx
// ✅ Method 1: Wrapping (implicit)
<label>
  Email
  <input type="email" name="email" />
</label>

// ✅ Method 2: htmlFor (explicit)
<label htmlFor="email">Email</label>
<input id="email" type="email" name="email" />

// ❌ Bad: No label association
<span>Email</span>
<input type="email" name="email" />
```

## Placeholders Are Not Labels

```jsx
// ❌ Bad: Placeholder as only label
<input placeholder="Enter your email" />

// ✅ Good: Real label with optional placeholder
<label htmlFor="email">Email</label>
<input 
  id="email" 
  placeholder="you@example.com"
/>
```

Placeholders disappear when typing, leaving users confused about what field they're in.

## Grouping with Fieldset

```jsx
<fieldset>
  <legend>Shipping Address</legend>
  
  <label htmlFor="street">Street</label>
  <input id="street" name="street" />
  
  <label htmlFor="city">City</label>
  <input id="city" name="city" />
  
  <label htmlFor="zip">ZIP Code</label>
  <input id="zip" name="zip" />
</fieldset>
```

## Radio Button Groups

```jsx
<fieldset>
  <legend>Preferred Contact Method</legend>
  
  <label>
    <input type="radio" name="contact" value="email" />
    Email
  </label>
  
  <label>
    <input type="radio" name="contact" value="phone" />
    Phone
  </label>
  
  <label>
    <input type="radio" name="contact" value="mail" />
    Mail
  </label>
</fieldset>
```

## Required Fields

```jsx
<label htmlFor="name">
  Name <span aria-hidden="true">*</span>
  <span className="sr-only">(required)</span>
</label>
<input 
  id="name" 
  name="name" 
  required 
  aria-required="true"
/>
```

Note: The visual asterisk is hidden from screen readers, which announce "required" from the attribute instead.

---

> 📘 *This lesson is part of the [React Form Mastery](https://stanza.dev/courses/react-forms) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*