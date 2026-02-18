---
source_course: "react"
source_lesson: "react-styled-components-intro"
---

# CSS-in-JS with Styled Components

Styled Components lets you write actual CSS in your JavaScript.

## Benefits

- **Dynamic styling**: Use props to change styles
- **Automatic vendor prefixing**: No need for autoprefixer
- **Scoped styles**: No class name bugs
- **Easy theming**: Built-in theme support

## Installation

```bash
npm install styled-components
```

## Code Examples

**Styled Components basics**

```tsx
import styled from 'styled-components';

// Basic styled component
const Button = styled.button`
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  
  /* Dynamic styles based on props */
  background-color: ${props => props.primary ? '#007bff' : '#6c757d'};
  color: white;
  
  &:hover {
    opacity: 0.9;
  }
  
  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
`;

// Extending styles
const LargeButton = styled(Button)`
  padding: 15px 30px;
  font-size: 20px;
`;

// Usage
function App() {
  return (
    <div>
      <Button>Secondary</Button>
      <Button primary>Primary</Button>
      <LargeButton primary>Large Primary</LargeButton>
    </div>
  );
}
```


## Resources

- [Styled Components Documentation](https://styled-components.com/docs) — Official styled-components docs

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*