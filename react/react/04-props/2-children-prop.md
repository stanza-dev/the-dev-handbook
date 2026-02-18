---
source_course: "react"
source_lesson: "react-children-prop"
---

# The `children` Prop

The `children` prop contains whatever you put between a component's opening and closing tags.

## Usage Patterns

1. **Wrapper components**: Cards, modals, layouts
2. **Composition**: Building flexible, reusable components
3. **Render props**: Passing functions as children

## Code Examples

**Modal component using children**

```tsx
function Modal({ isOpen, onClose, children }) {
  if (!isOpen) return null;
  
  return (
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal" onClick={e => e.stopPropagation()}>
        <button className="modal-close" onClick={onClose}>
          ×
        </button>
        {children}
      </div>
    </div>
  );
}

// Usage
<Modal isOpen={showModal} onClose={() => setShowModal(false)}>
  <h2>Modal Title</h2>
  <p>Modal content goes here!</p>
</Modal>
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*