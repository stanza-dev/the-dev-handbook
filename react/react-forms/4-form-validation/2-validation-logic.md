---
source_course: "react-forms"
source_lesson: "react-forms-building-validation-logic"
---

# Building Validation Logic

Let's build a complete validation system from scratch.

## Validation Functions

```jsx
// Individual field validators
const validators = {
  required: (value) => 
    !value?.trim() ? 'This field is required' : null,
  
  email: (value) => {
    if (!value) return null;
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return !emailRegex.test(value) ? 'Invalid email address' : null;
  },
  
  minLength: (min) => (value) => {
    if (!value) return null;
    return value.length < min 
      ? `Must be at least ${min} characters` 
      : null;
  },
  
  maxLength: (max) => (value) => {
    if (!value) return null;
    return value.length > max 
      ? `Must be no more than ${max} characters` 
      : null;
  },
  
  matches: (pattern, message) => (value) => {
    if (!value) return null;
    return !pattern.test(value) ? message : null;
  }
};
```

## Field Configuration

```jsx
const fieldConfig = {
  email: [
    validators.required,
    validators.email
  ],
  password: [
    validators.required,
    validators.minLength(8),
    validators.matches(
      /[A-Z]/,
      'Must contain at least one uppercase letter'
    ),
    validators.matches(
      /[0-9]/,
      'Must contain at least one number'
    )
  ],
  username: [
    validators.required,
    validators.minLength(3),
    validators.maxLength(20)
  ]
};
```

## Validate Single Field

```jsx
function validateField(name, value) {
  const fieldValidators = fieldConfig[name] || [];
  
  for (const validate of fieldValidators) {
    const error = validate(value);
    if (error) return error; // Return first error
  }
  
  return null;
}
```

## Validate Entire Form

```jsx
function validateForm(formData) {
  const errors = {};
  
  for (const [field, value] of Object.entries(formData)) {
    const error = validateField(field, value);
    if (error) {
      errors[field] = error;
    }
  }
  
  return errors;
}
```

## Complete Example

```jsx
function SignupForm() {
  const [formData, setFormData] = useState({
    email: '',
    password: '',
    username: ''
  });
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});

  function handleChange(e) {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    
    if (touched[name]) {
      setErrors(prev => ({
        ...prev,
        [name]: validateField(name, value)
      }));
    }
  }

  function handleBlur(e) {
    const { name, value } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
    setErrors(prev => ({
      ...prev,
      [name]: validateField(name, value)
    }));
  }

  function handleSubmit(e) {
    e.preventDefault();
    const formErrors = validateForm(formData);
    setErrors(formErrors);
    setTouched({
      email: true,
      password: true,
      username: true
    });
    
    if (Object.keys(formErrors).length === 0) {
      console.log('Form is valid!', formData);
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          name="email"
          value={formData.email}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.email && errors.email && (
          <span className="error">{errors.email}</span>
        )}
      </div>
      {/* More fields... */}
    </form>
  );
}
```

---

> 📘 *This lesson is part of the [React Form Mastery](https://stanza.dev/courses/react-forms) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*