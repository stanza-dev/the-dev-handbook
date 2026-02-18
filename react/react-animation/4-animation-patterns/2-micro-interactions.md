---
source_course: "react-animation"
source_lesson: "react-animation-micro-interactions"
---

# Micro-interactions

Subtle animations that enhance user experience.

## Like Button

```jsx
function LikeButton() {
  const [isLiked, setIsLiked] = useState(false);

  return (
    <motion.button
      onClick={() => setIsLiked(!isLiked)}
      whileTap={{ scale: 0.9 }}
    >
      <motion.svg
        viewBox="0 0 24 24"
        animate={{
          scale: isLiked ? [1, 1.3, 1] : 1,
          fill: isLiked ? '#ef4444' : 'none',
        }}
        transition={{ duration: 0.3 }}
      >
        <path
          d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"
          stroke="currentColor"
          strokeWidth="2"
        />
      </motion.svg>
    </motion.button>
  );
}
```

## Toggle Switch

```jsx
function Toggle({ isOn, onToggle }) {
  return (
    <motion.button
      className="toggle-track"
      animate={{ backgroundColor: isOn ? '#22c55e' : '#e5e7eb' }}
      onClick={onToggle}
    >
      <motion.div
        className="toggle-thumb"
        layout
        transition={{
          type: 'spring',
          stiffness: 700,
          damping: 30,
        }}
        style={{ marginLeft: isOn ? 'auto' : 0 }}
      />
    </motion.button>
  );
}
```

## Checkbox

```jsx
function Checkbox({ isChecked, onChange }) {
  return (
    <motion.button
      className="checkbox"
      animate={{
        backgroundColor: isChecked ? '#3b82f6' : '#ffffff',
        borderColor: isChecked ? '#3b82f6' : '#d1d5db',
      }}
      onClick={onChange}
    >
      <motion.svg
        viewBox="0 0 24 24"
        initial={false}
        animate={{ pathLength: isChecked ? 1 : 0 }}
      >
        <motion.path
          d="M5 12l5 5L20 7"
          fill="none"
          stroke="white"
          strokeWidth="3"
          strokeLinecap="round"
          strokeLinejoin="round"
        />
      </motion.svg>
    </motion.button>
  );
}
```

## Notification Badge

```jsx
function NotificationBadge({ count }) {
  return (
    <AnimatePresence>
      {count > 0 && (
        <motion.span
          key="badge"
          initial={{ scale: 0 }}
          animate={{ scale: 1 }}
          exit={{ scale: 0 }}
          transition={{ type: 'spring', stiffness: 500, damping: 25 }}
          className="badge"
        >
          {count}
        </motion.span>
      )}
    </AnimatePresence>
  );
}
```

## Copy Button Feedback

```jsx
function CopyButton({ text }) {
  const [copied, setCopied] = useState(false);

  const handleCopy = async () => {
    await navigator.clipboard.writeText(text);
    setCopied(true);
    setTimeout(() => setCopied(false), 2000);
  };

  return (
    <motion.button
      onClick={handleCopy}
      whileTap={{ scale: 0.95 }}
    >
      <AnimatePresence mode="wait">
        {copied ? (
          <motion.span
            key="check"
            initial={{ opacity: 0, y: 10 }}
            animate={{ opacity: 1, y: 0 }}
            exit={{ opacity: 0, y: -10 }}
          >
            ✓ Copied!
          </motion.span>
        ) : (
          <motion.span
            key="copy"
            initial={{ opacity: 0, y: 10 }}
            animate={{ opacity: 1, y: 0 }}
            exit={{ opacity: 0, y: -10 }}
          >
            Copy
          </motion.span>
        )}
      </AnimatePresence>
    </motion.button>
  );
}
```

## Floating Action Button

```jsx
function FAB({ actions }) {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div className="fab-container">
      <AnimatePresence>
        {isOpen && actions.map((action, index) => (
          <motion.button
            key={action.id}
            className="fab-action"
            initial={{ opacity: 0, y: 20 }}
            animate={{ opacity: 1, y: 0 }}
            exit={{ opacity: 0, y: 20 }}
            transition={{ delay: index * 0.05 }}
            onClick={action.onClick}
          >
            {action.icon}
          </motion.button>
        ))}
      </AnimatePresence>
      
      <motion.button
        className="fab-main"
        animate={{ rotate: isOpen ? 45 : 0 }}
        onClick={() => setIsOpen(!isOpen)}
      >
        +
      </motion.button>
    </div>
  );
}
```

---

> 📘 *This lesson is part of the [React Animation & Motion](https://stanza.dev/courses/react-animation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*