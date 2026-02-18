---
source_course: "react-forms"
source_lesson: "react-forms-checkboxes-and-radios"
---

# Checkboxes & Radio Buttons

Checkboxes and radio buttons work differently from text inputs - they use `checked` instead of `value`.

## Single Checkbox

For a single boolean value:

```jsx
import { useState } from 'react';

function Newsletter() {
  const [subscribed, setSubscribed] = useState(false);

  return (
    <label>
      <input
        type="checkbox"
        checked={subscribed}
        onChange={(e) => setSubscribed(e.target.checked)}
      />
      Subscribe to newsletter
    </label>
  );
}
```

Note: We use `e.target.checked` (boolean), not `e.target.value`.

## Multiple Checkboxes

For selecting multiple options, use an array or object:

```jsx
function InterestsForm() {
  const [interests, setInterests] = useState({
    sports: false,
    music: false,
    reading: false,
    gaming: false
  });

  function handleCheckbox(e) {
    const { name, checked } = e.target;
    setInterests(prev => ({
      ...prev,
      [name]: checked
    }));
  }

  return (
    <fieldset>
      <legend>Select your interests:</legend>
      {Object.keys(interests).map(interest => (
        <label key={interest}>
          <input
            type="checkbox"
            name={interest}
            checked={interests[interest]}
            onChange={handleCheckbox}
          />
          {interest.charAt(0).toUpperCase() + interest.slice(1)}
        </label>
      ))}
    </fieldset>
  );
}
```

## Radio Buttons

Radio buttons are for selecting ONE option from many:

```jsx
function SizeSelector() {
  const [size, setSize] = useState('medium');

  return (
    <fieldset>
      <legend>Select size:</legend>
      {['small', 'medium', 'large'].map(option => (
        <label key={option}>
          <input
            type="radio"
            name="size"
            value={option}
            checked={size === option}
            onChange={(e) => setSize(e.target.value)}
          />
          {option.charAt(0).toUpperCase() + option.slice(1)}
        </label>
      ))}
    </fieldset>
  );
}
```

All radio buttons in a group share the same `name` attribute. The `checked` prop determines which one is selected.

📚 **Learn more**: [Updating Objects in State](https://react.dev/learn/updating-objects-in-state)

---

> 📘 *This lesson is part of the [React Form Mastery](https://stanza.dev/courses/react-forms) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*