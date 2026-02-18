---
source_course: "react-forms"
source_lesson: "react-forms-single-state-object"
---

# Single State Object Pattern

When forms have multiple fields, managing individual `useState` calls becomes unwieldy. A better approach is using a single state object.

## The Problem with Multiple useState

```jsx
// ❌ Gets messy with many fields
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const [email, setEmail] = useState('');
const [phone, setPhone] = useState('');
const [address, setAddress] = useState('');
const [city, setCity] = useState('');
// ... and more
```

## Single Object Solution

```jsx
import { useState } from 'react';

function RegistrationForm() {
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: '',
    phone: '',
    address: '',
    city: ''
  });

  // Generic handler for all inputs
  function handleChange(e) {
    const { name, value, type, checked } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  }

  function handleSubmit(e) {
    e.preventDefault();
    console.log(formData);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="firstName"
        value={formData.firstName}
        onChange={handleChange}
        placeholder="First Name"
      />
      <input
        name="lastName"
        value={formData.lastName}
        onChange={handleChange}
        placeholder="Last Name"
      />
      {/* More inputs... */}
      <button type="submit">Register</button>
    </form>
  );
}
```

## Benefits

1. **Single handler**: One `handleChange` for all inputs
2. **Easy reset**: `setFormData(initialState)`
3. **Easy submit**: All data in one object
4. **Computed property names**: `[name]: value` maps input name to state key

## Resetting the Form

```jsx
const initialState = {
  firstName: '',
  lastName: '',
  email: ''
};

function Form() {
  const [formData, setFormData] = useState(initialState);

  function handleReset() {
    setFormData(initialState);
  }

  // ...
}
```

📚 **Learn more**: [Choosing the State Structure](https://react.dev/learn/choosing-the-state-structure)

---

> 📘 *This lesson is part of the [React Form Mastery](https://stanza.dev/courses/react-forms) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*