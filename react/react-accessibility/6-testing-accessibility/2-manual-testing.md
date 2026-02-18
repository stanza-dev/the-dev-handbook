---
source_course: "react-accessibility"
source_lesson: "react-accessibility-manual-testing"
---

# Manual Accessibility Testing

Automated tests catch ~30% of issues. Manual testing is essential.

## Keyboard Testing

1. **Unplug your mouse**
2. Navigate using only keyboard
3. Check:
   - Can you reach all interactive elements with Tab?
   - Is focus visible at all times?
   - Can you activate buttons with Enter/Space?
   - Can you escape modals with Escape?
   - Does focus move logically?

## Screen Reader Testing

### VoiceOver (macOS)
```
Cmd + F5 to enable
Ctrl + Option + Right Arrow to navigate
```

### NVDA (Windows, Free)
```
Download from nvaccess.org
Insert + Down Arrow to read continuously
```

### Testing Checklist

- [ ] Page title announced
- [ ] Headings navigable
- [ ] Images have descriptions
- [ ] Forms have labels
- [ ] Errors announced
- [ ] Dynamic content announced
- [ ] Links/buttons have clear names

## Browser DevTools

### Chrome Accessibility Inspector

1. DevTools → Elements
2. Accessibility tab
3. Check computed accessibility tree

### Firefox Accessibility Inspector

1. DevTools → Accessibility tab
2. "Check for issues"
3. View full accessibility tree

## Color Contrast

### Browser Extensions
- WAVE Evaluation Tool
- axe DevTools
- Accessibility Insights

### Requirements (WCAG AA)
- Normal text: 4.5:1 ratio
- Large text (18px+): 3:1 ratio
- UI components: 3:1 ratio

## Zoom Testing

1. Zoom to 200% (Cmd/Ctrl + +)
2. Check:
   - No horizontal scrolling
   - Text is readable
   - No content cut off
   - Interactive elements still usable

## Testing Checklist

```markdown
## Keyboard
- [ ] All interactive elements focusable
- [ ] Focus indicator visible
- [ ] Logical tab order
- [ ] No keyboard traps
- [ ] Skip link works

## Screen Reader
- [ ] Page has title
- [ ] Landmarks present
- [ ] Headings hierarchical
- [ ] Images described
- [ ] Forms labeled
- [ ] Errors announced

## Visual
- [ ] Sufficient color contrast
- [ ] Works at 200% zoom
- [ ] Not color-only information
- [ ] Focus visible
```

---

> 📘 *This lesson is part of the [React Accessibility (a11y)](https://stanza.dev/courses/react-accessibility) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*