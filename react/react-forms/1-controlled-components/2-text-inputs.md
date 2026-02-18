---
source_course: "react-forms"
source_lesson: "react-forms-handling-text-inputs"
---

# Handling Text Inputs

Text inputs are the most common form elements. Let's explore how to handle different text input types in React.

## Single Text Input

```jsx
import { useState } from 'react';

function EmailForm() {
  const [email, setEmail] = useState('');

  return (
    <label>
      Email:
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="you@example.com"
      />
    </label>
  );
}
```

## Multiple Text Inputs

When you have multiple inputs, you can use an object to store all values:

```jsx
import { useState } from 'react';

function SignupForm() {
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: ''
  });

  function handleChange(e) {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value  // Computed property name
    }));
  }

  return (
    <form>
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
      <input
        name="email"
        type="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
    </form>
  );
}
```

## Textarea

In HTML, `<textarea>` uses children for its value. In React, we use the `value` attribute:

```jsx
function CommentBox() {
  const [comment, setComment] = useState('');

  return (
    <textarea
      value={comment}
      onChange={(e) => setComment(e.target.value)}
      rows={4}
      placeholder="Write your comment..."
    />
  );
}
```

## Password Inputs

```jsx
function PasswordInput() {
  const [password, setPassword] = useState('');
  const [showPassword, setShowPassword] = useState(false);

  return (
    <div>
      <input
        type={showPassword ? 'text' : 'password'}
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      <button
        type="button"
        onClick={() => setShowPassword(!showPassword)}
      >
        {showPassword ? 'Hide' : 'Show'}
      </button>
    </div>
  );
}
```

📚 **Learn more**: [State: A Component's Memory](https://react.dev/learn/state-a-components-memory)

---

> 📘 *This lesson is part of the [React Form Mastery](https://stanza.dev/courses/react-forms) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*