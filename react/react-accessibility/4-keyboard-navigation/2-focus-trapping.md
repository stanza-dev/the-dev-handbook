---
source_course: "react-accessibility"
source_lesson: "react-accessibility-focus-trapping"
---

# Focus Trapping

Modals and dialogs must trap focus inside them.

## Why Focus Trapping?

When a modal opens:
1. Focus should move to the modal
2. Tab should cycle within the modal
3. Focus shouldn't escape to content behind
4. On close, focus returns to trigger

## Basic Focus Trap Hook

```jsx
import { useEffect, useRef } from 'react';

function useFocusTrap(isActive) {
  const containerRef = useRef(null);
  const previousFocusRef = useRef(null);

  useEffect(() => {
    if (!isActive) return;

    // Save current focus
    previousFocusRef.current = document.activeElement;

    const container = containerRef.current;
    if (!container) return;

    // Find focusable elements
    const focusableSelector = 
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])';
    const focusableElements = 
      container.querySelectorAll(focusableSelector);
    const firstElement = focusableElements[0];
    const lastElement = focusableElements[focusableElements.length - 1];

    // Focus first element
    firstElement?.focus();

    // Handle Tab key
    const handleKeyDown = (e) => {
      if (e.key !== 'Tab') return;

      if (e.shiftKey && document.activeElement === firstElement) {
        e.preventDefault();
        lastElement?.focus();
      } else if (!e.shiftKey && document.activeElement === lastElement) {
        e.preventDefault();
        firstElement?.focus();
      }
    };

    container.addEventListener('keydown', handleKeyDown);

    return () => {
      container.removeEventListener('keydown', handleKeyDown);
      // Restore focus
      previousFocusRef.current?.focus();
    };
  }, [isActive]);

  return containerRef;
}
```

## Usage in Modal

```jsx
function Modal({ isOpen, onClose, children }) {
  const modalRef = useFocusTrap(isOpen);

  if (!isOpen) return null;

  return (
    <div className="modal-overlay" onClick={onClose}>
      <div
        ref={modalRef}
        role="dialog"
        aria-modal="true"
        onClick={(e) => e.stopPropagation()}
      >
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>
  );
}
```

## Using a Library

Consider `focus-trap-react` for production:

```jsx
import FocusTrap from 'focus-trap-react';

function Modal({ isOpen, onClose, children }) {
  if (!isOpen) return null;

  return (
    <FocusTrap>
      <div role="dialog" aria-modal="true">
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </FocusTrap>
  );
}
```

## inert Attribute (Modern)

Hide everything behind modal:

```jsx
function App() {
  const [modalOpen, setModalOpen] = useState(false);

  return (
    <>
      <div inert={modalOpen ? '' : undefined}>
        {/* Main app content */}
      </div>
      {modalOpen && <Modal onClose={() => setModalOpen(false)} />}
    </>
  );
}
```

---

> 📘 *This lesson is part of the [React Accessibility (a11y)](https://stanza.dev/courses/react-accessibility) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*