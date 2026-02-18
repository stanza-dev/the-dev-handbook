---
source_course: "react-forms"
source_lesson: "react-forms-nested-form-state"
---

# Nested Form State

Real-world forms often have nested data structures. Let's learn how to handle them properly.

## Example: Address Form

```jsx
function ProfileForm() {
  const [profile, setProfile] = useState({
    name: '',
    email: '',
    address: {
      street: '',
      city: '',
      state: '',
      zipCode: ''
    },
    preferences: {
      newsletter: false,
      notifications: true
    }
  });

  // Handle top-level fields
  function handleChange(e) {
    const { name, value } = e.target;
    setProfile(prev => ({
      ...prev,
      [name]: value
    }));
  }

  // Handle nested address fields
  function handleAddressChange(e) {
    const { name, value } = e.target;
    setProfile(prev => ({
      ...prev,
      address: {
        ...prev.address,
        [name]: value
      }
    }));
  }

  // Handle nested preference fields
  function handlePreferenceChange(e) {
    const { name, checked } = e.target;
    setProfile(prev => ({
      ...prev,
      preferences: {
        ...prev.preferences,
        [name]: checked
      }
    }));
  }

  return (
    <form>
      <input
        name="name"
        value={profile.name}
        onChange={handleChange}
      />
      
      <fieldset>
        <legend>Address</legend>
        <input
          name="street"
          value={profile.address.street}
          onChange={handleAddressChange}
        />
        <input
          name="city"
          value={profile.address.city}
          onChange={handleAddressChange}
        />
      </fieldset>
      
      <fieldset>
        <legend>Preferences</legend>
        <label>
          <input
            type="checkbox"
            name="newsletter"
            checked={profile.preferences.newsletter}
            onChange={handlePreferenceChange}
          />
          Newsletter
        </label>
      </fieldset>
    </form>
  );
}
```

## Generic Deep Update Helper

For deeply nested state, a helper function can be useful:

```jsx
function updateNestedState(obj, path, value) {
  const keys = path.split('.');
  const lastKey = keys.pop();
  
  const newObj = { ...obj };
  let current = newObj;
  
  for (const key of keys) {
    current[key] = { ...current[key] };
    current = current[key];
  }
  
  current[lastKey] = value;
  return newObj;
}

// Usage
setProfile(prev => 
  updateNestedState(prev, 'address.city', 'New York')
);
```

📚 **Learn more**: [Updating Objects in State](https://react.dev/learn/updating-objects-in-state)

---

> 📘 *This lesson is part of the [React Form Mastery](https://stanza.dev/courses/react-forms) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*