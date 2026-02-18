---
source_course: "react-accessibility"
source_lesson: "react-accessibility-accessible-tabs"
---

# Accessible Tabs

Tabs require specific ARIA roles and keyboard navigation.

## ARIA Roles for Tabs

- `tablist`: Container for tabs
- `tab`: Each tab button
- `tabpanel`: Content panel

## Complete Tabs Component

```jsx
import { useState, useRef } from 'react';

function Tabs({ tabs }) {
  const [activeIndex, setActiveIndex] = useState(0);
  const tabRefs = useRef([]);

  const handleKeyDown = (e, index) => {
    let newIndex = index;

    switch (e.key) {
      case 'ArrowRight':
        newIndex = (index + 1) % tabs.length;
        break;
      case 'ArrowLeft':
        newIndex = (index - 1 + tabs.length) % tabs.length;
        break;
      case 'Home':
        newIndex = 0;
        break;
      case 'End':
        newIndex = tabs.length - 1;
        break;
      default:
        return;
    }

    e.preventDefault();
    setActiveIndex(newIndex);
    tabRefs.current[newIndex]?.focus();
  };

  return (
    <div>
      <div role="tablist" aria-label="Content tabs">
        {tabs.map((tab, index) => (
          <button
            key={tab.id}
            ref={(el) => (tabRefs.current[index] = el)}
            role="tab"
            id={`tab-${tab.id}`}
            aria-selected={index === activeIndex}
            aria-controls={`panel-${tab.id}`}
            tabIndex={index === activeIndex ? 0 : -1}
            onClick={() => setActiveIndex(index)}
            onKeyDown={(e) => handleKeyDown(e, index)}
          >
            {tab.label}
          </button>
        ))}
      </div>

      {tabs.map((tab, index) => (
        <div
          key={tab.id}
          role="tabpanel"
          id={`panel-${tab.id}`}
          aria-labelledby={`tab-${tab.id}`}
          hidden={index !== activeIndex}
          tabIndex={0}
        >
          {tab.content}
        </div>
      ))}
    </div>
  );
}
```

## Usage

```jsx
function App() {
  const tabs = [
    { id: 'profile', label: 'Profile', content: <ProfilePanel /> },
    { id: 'settings', label: 'Settings', content: <SettingsPanel /> },
    { id: 'notifications', label: 'Notifications', content: <NotificationsPanel /> },
  ];

  return <Tabs tabs={tabs} />;
}
```

## Keyboard Navigation

| Key | Action |
|-----|--------|
| Arrow Right | Next tab |
| Arrow Left | Previous tab |
| Home | First tab |
| End | Last tab |
| Tab | Move to panel |

## Styling Considerations

```css
[role="tab"] {
  /* Remove default button styling */
}

[role="tab"][aria-selected="true"] {
  /* Active tab styling */
  border-bottom: 2px solid blue;
}

[role="tab"]:focus-visible {
  /* Clear focus indicator */
  outline: 2px solid blue;
}
```

---

> 📘 *This lesson is part of the [React Accessibility (a11y)](https://stanza.dev/courses/react-accessibility) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*