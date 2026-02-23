---
source_course: "django-security"
source_lesson: "django-security-ui-redress-attacks"
---

# Understanding UI Redress Attacks

## Introduction

Clickjacking is just one form of UI redress attack. Understanding the variations helps you implement comprehensive protection.

## Key Concepts

**Clickjacking**: Tricking users into clicking invisible elements.

**Likejacking**: Variant that targets social media like buttons.

**Cursorjacking**: Moving the visible cursor away from the real cursor.

**Overlay Attacks**: Covering legitimate UI with fake elements.



## Real World Context

In 2008, researchers demonstrated a clickjacking attack that could enable a user's webcam through Adobe Flash by tricking them into clicking through a series of invisible dialogs. More recently, multi-step clickjacking attacks have been used to change OAuth permissions and authorize malicious applications.

## Deep Dive

### Attack Variations

**Classic Clickjacking**:
```html
<iframe src="https://bank.com/transfer" style="opacity:0"></iframe>
<button style="position:absolute">Win Prize!</button>
```

**Likejacking**:
```html
<div style="position:relative">
    <button>Download Free Music</button>
    <iframe src="https://facebook.com/like?page=spam" 
            style="position:absolute; opacity:0"></iframe>
</div>
```

**Multi-Click Attacks**:
```javascript
// Move iframe to follow cursor
document.onmousemove = function(e) {
    iframe.style.left = e.pageX + 'px';
    iframe.style.top = e.pageY + 'px';
};
```

### Defense in Depth

```python
# 1. HTTP Headers
X_FRAME_OPTIONS = 'DENY'
CSP_FRAME_ANCESTORS = ("'none'",)

# 2. Sensitive actions require confirmation
def transfer_money(request):
    if request.method == 'POST':
        # Require re-authentication for sensitive actions
        if not request.session.get('recently_authenticated'):
            return redirect('confirm_identity')

# 3. CAPTCHA for automated attack prevention
```

### JavaScript Frame Detection

```html
<script>
// Detect if we're in a frame
if (window.self !== window.top) {
    // Log potential attack
    fetch('/api/security/frame-detected/', {
        method: 'POST',
        body: JSON.stringify({
            referrer: document.referrer,
            parent_url: window.parent.location.href // may fail
        })
    }).catch(() => {});
    
    // Break out or hide content
    document.body.innerHTML = 'This page cannot be framed.';
}
</script>
```



## Common Pitfalls

1. **Relying solely on HTTP headers** — Headers prevent framing but not other UI redress techniques like overlay attacks within the same page; add confirmation dialogs for sensitive actions.
2. **Trusting JavaScript frame-busting code** — Attackers can neutralize frame-busting scripts using iframe sandbox attributes or the onbeforeunload event; always use HTTP headers as the primary defense.

## Best Practices

1. **Layer your defenses**: Headers + confirmation + CAPTCHA.
2. **Re-authenticate for sensitive actions**: Extra verification.
3. **Monitor for attacks**: Log frame detection attempts.

## Summary

UI redress attacks go beyond simple clickjacking. Protect with HTTP headers, require confirmation for sensitive actions, and implement monitoring to detect attack attempts.

## Resources

- [OWASP Clickjacking](https://owasp.org/www-community/attacks/Clickjacking) — OWASP Clickjacking prevention

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*