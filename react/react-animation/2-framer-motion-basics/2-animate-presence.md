---
source_course: "react-animation"
source_lesson: "react-animation-animate-presence"
---

# AnimatePresence for Exit Animations

`AnimatePresence` enables exit animations when components unmount.

## Basic Usage

```jsx
import { motion, AnimatePresence } from 'framer-motion';

function Modal({ isOpen, onClose }) {
  return (
    <AnimatePresence>
      {isOpen && (
        <motion.div
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
          className="modal-overlay"
          onClick={onClose}
        >
          <motion.div
            initial={{ scale: 0.9, opacity: 0 }}
            animate={{ scale: 1, opacity: 1 }}
            exit={{ scale: 0.9, opacity: 0 }}
            className="modal-content"
            onClick={(e) => e.stopPropagation()}
          >
            Modal Content
          </motion.div>
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

## Animating Lists

```jsx
function TodoList({ items, onRemove }) {
  return (
    <ul>
      <AnimatePresence>
        {items.map((item) => (
          <motion.li
            key={item.id}
            initial={{ opacity: 0, height: 0 }}
            animate={{ opacity: 1, height: 'auto' }}
            exit={{ opacity: 0, height: 0 }}
            transition={{ duration: 0.2 }}
          >
            {item.text}
            <button onClick={() => onRemove(item.id)}>×</button>
          </motion.li>
        ))}
      </AnimatePresence>
    </ul>
  );
}
```

## Mode: Wait vs Sync

```jsx
// wait: Exit finishes before enter starts
<AnimatePresence mode="wait">
  <motion.div key={currentPage}>
    {currentPage}
  </motion.div>
</AnimatePresence>

// sync: Exit and enter animate together (default)
<AnimatePresence mode="sync">
  {/* ... */}
</AnimatePresence>
```

## Page Transitions

```jsx
import { useLocation } from 'react-router-dom';

function App() {
  const location = useLocation();

  return (
    <AnimatePresence mode="wait">
      <motion.main
        key={location.pathname}
        initial={{ opacity: 0, x: 20 }}
        animate={{ opacity: 1, x: 0 }}
        exit={{ opacity: 0, x: -20 }}
        transition={{ duration: 0.3 }}
      >
        <Routes location={location}>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
        </Routes>
      </motion.main>
    </AnimatePresence>
  );
}
```

## onExitComplete Callback

```jsx
function Wizard({ step, onComplete }) {
  return (
    <AnimatePresence
      mode="wait"
      onExitComplete={() => {
        // Called after exit animation completes
        window.scrollTo(0, 0);
      }}
    >
      <motion.div key={step}>
        {/* Step content */}
      </motion.div>
    </AnimatePresence>
  );
}
```

## Notification Toast

```jsx
function ToastContainer({ toasts, removeToast }) {
  return (
    <div className="toast-container">
      <AnimatePresence>
        {toasts.map((toast) => (
          <motion.div
            key={toast.id}
            initial={{ opacity: 0, y: 50, scale: 0.3 }}
            animate={{ opacity: 1, y: 0, scale: 1 }}
            exit={{ opacity: 0, scale: 0.5, transition: { duration: 0.2 } }}
            className="toast"
          >
            {toast.message}
            <button onClick={() => removeToast(toast.id)}>×</button>
          </motion.div>
        ))}
      </AnimatePresence>
    </div>
  );
}
```

---

> 📘 *This lesson is part of the [React Animation & Motion](https://stanza.dev/courses/react-animation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*