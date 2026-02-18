---
source_course: "react-accessibility"
source_lesson: "react-accessibility-assistive-technologies"
---

# Understanding Assistive Technologies

To build accessible apps, understand how people use them.

## Screen Readers

Software that reads content aloud:

- **NVDA** (Windows, free)
- **JAWS** (Windows, commercial)
- **VoiceOver** (macOS/iOS, built-in)
- **TalkBack** (Android, built-in)

### How Screen Readers Work

1. Parse the DOM and accessibility tree
2. Announce content and structure
3. Navigate via headings, landmarks, links
4. Interact with forms and controls

### What Screen Readers Announce

```html
<!-- Announces: "Submit, button" -->
<button>Submit</button>

<!-- Announces: "Email, edit text" -->
<label for="email">Email</label>
<input id="email" type="email" />

<!-- Announces: "Profile picture, image" -->
<img src="profile.jpg" alt="Profile picture" />
```

## Keyboard Navigation

Many users navigate entirely by keyboard:

| Key | Action |
|-----|--------|
| Tab | Move to next focusable element |
| Shift+Tab | Move to previous element |
| Enter | Activate buttons, links |
| Space | Activate buttons, checkboxes |
| Arrow keys | Navigate within components |
| Escape | Close dialogs, cancel |

## Voice Control

- Dragon NaturallySpeaking
- Voice Control (macOS/iOS)
- Voice Access (Android)

Users speak commands like:
- "Click Submit"
- "Show numbers" (overlay numbers on clickable elements)
- "Press Tab"

## Screen Magnifiers

- ZoomText
- Built-in OS zoom
- Browser zoom

Considerations:
- Content should reflow at high zoom
- Don't break at 200-400% zoom

## Alternative Input Devices

- Switch devices (single button)
- Eye tracking
- Head tracking
- Sip-and-puff devices

## Testing Your App

```bash
# Try navigating with keyboard only
# No mouse!

# Turn on VoiceOver (macOS)
# Cmd + F5

# Try with screen reader
# Listen to how your content sounds
```

---

> 📘 *This lesson is part of the [React Accessibility (a11y)](https://stanza.dev/courses/react-accessibility) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*