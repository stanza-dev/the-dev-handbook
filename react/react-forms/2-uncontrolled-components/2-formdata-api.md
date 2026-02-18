---
source_course: "react-forms"
source_lesson: "react-forms-using-formdata-api"
---

# Using the FormData API

The **FormData** API provides a way to construct a set of key/value pairs representing form fields and their values. This is perfect for uncontrolled forms!

## Basic FormData Usage

```jsx
function ContactForm() {
  function handleSubmit(e) {
    e.preventDefault();
    
    // Create FormData from the form element
    const formData = new FormData(e.target);
    
    // Get individual values
    const name = formData.get('name');
    const email = formData.get('email');
    
    console.log({ name, email });
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" placeholder="Name" />
      <input name="email" type="email" placeholder="Email" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

## Converting to Object

To get all form data as a plain object:

```jsx
function handleSubmit(e) {
  e.preventDefault();
  const formData = new FormData(e.target);
  
  // Convert to plain object
  const data = Object.fromEntries(formData.entries());
  console.log(data); // { name: '...', email: '...' }
}
```

## Sending to an API

FormData can be sent directly as a fetch body:

```jsx
async function handleSubmit(e) {
  e.preventDefault();
  const formData = new FormData(e.target);
  
  // Send directly - works for multipart/form-data
  await fetch('/api/contact', {
    method: 'POST',
    body: formData
  });
  
  // OR convert to JSON
  await fetch('/api/contact', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(Object.fromEntries(formData))
  });
}
```

## Handling Checkboxes

Checkboxes only appear in FormData if they're checked:

```jsx
<form onSubmit={handleSubmit}>
  <label>
    <input type="checkbox" name="newsletter" value="yes" />
    Subscribe
  </label>
</form>

// If checked: formData.get('newsletter') → 'yes'
// If unchecked: formData.get('newsletter') → null
```

FormData is especially useful in React 19 with form actions, as we'll see later!

---

> 📘 *This lesson is part of the [React Form Mastery](https://stanza.dev/courses/react-forms) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*