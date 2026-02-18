---
source_course: "react-accessibility"
source_lesson: "react-accessibility-accessible-dropdown"
---

# Accessible Dropdown

Accessible select/dropdown with custom styling.

## Custom Select Component

```jsx
import { useState, useRef, useEffect } from 'react';

function Select({ options, value, onChange, label }) {
  const [isOpen, setIsOpen] = useState(false);
  const [highlightedIndex, setHighlightedIndex] = useState(0);
  const containerRef = useRef(null);
  const listRef = useRef(null);

  const selectedOption = options.find(o => o.value === value);

  // Close on outside click
  useEffect(() => {
    const handleClickOutside = (e) => {
      if (!containerRef.current?.contains(e.target)) {
        setIsOpen(false);
      }
    };
    document.addEventListener('click', handleClickOutside);
    return () => document.removeEventListener('click', handleClickOutside);
  }, []);

  const handleKeyDown = (e) => {
    switch (e.key) {
      case 'Enter':
      case ' ':
        e.preventDefault();
        if (isOpen) {
          onChange(options[highlightedIndex].value);
          setIsOpen(false);
        } else {
          setIsOpen(true);
        }
        break;
      case 'ArrowDown':
        e.preventDefault();
        if (!isOpen) {
          setIsOpen(true);
        } else {
          setHighlightedIndex(i => 
            Math.min(i + 1, options.length - 1)
          );
        }
        break;
      case 'ArrowUp':
        e.preventDefault();
        setHighlightedIndex(i => Math.max(i - 1, 0));
        break;
      case 'Escape':
        setIsOpen(false);
        break;
    }
  };

  return (
    <div ref={containerRef} className="select-container">
      <label id="select-label">{label}</label>
      <button
        type="button"
        role="combobox"
        aria-labelledby="select-label"
        aria-haspopup="listbox"
        aria-expanded={isOpen}
        aria-controls="select-listbox"
        aria-activedescendant={
          isOpen ? `option-${highlightedIndex}` : undefined
        }
        onClick={() => setIsOpen(!isOpen)}
        onKeyDown={handleKeyDown}
      >
        {selectedOption?.label || 'Select...'}
      </button>

      {isOpen && (
        <ul
          ref={listRef}
          id="select-listbox"
          role="listbox"
          aria-labelledby="select-label"
        >
          {options.map((option, index) => (
            <li
              key={option.value}
              id={`option-${index}`}
              role="option"
              aria-selected={option.value === value}
              className={index === highlightedIndex ? 'highlighted' : ''}
              onClick={() => {
                onChange(option.value);
                setIsOpen(false);
              }}
              onMouseEnter={() => setHighlightedIndex(index)}
            >
              {option.label}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

## Usage

```jsx
function Form() {
  const [country, setCountry] = useState('');

  const countries = [
    { value: 'us', label: 'United States' },
    { value: 'uk', label: 'United Kingdom' },
    { value: 'ca', label: 'Canada' },
  ];

  return (
    <Select
      label="Country"
      options={countries}
      value={country}
      onChange={setCountry}
    />
  );
}
```

## Consider Native Select

```jsx
// Often the most accessible choice!
<label htmlFor="country">Country</label>
<select id="country" value={value} onChange={handleChange}>
  {options.map(opt => (
    <option key={opt.value} value={opt.value}>
      {opt.label}
    </option>
  ))}
</select>
```

---

> 📘 *This lesson is part of the [React Accessibility (a11y)](https://stanza.dev/courses/react-accessibility) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*