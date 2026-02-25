---
source_course: "php-essentials"
source_lesson: "php-essentials-handling-forms"
---

# Handling Form Submissions

## Introduction

Forms are the primary mechanism through which users send data to a PHP application. Whether it is a login page, a contact form, or a search bar, understanding how to receive, validate, and process form data is one of the most essential PHP skills. This lesson covers HTTP methods, superglobal arrays, and patterns for handling form submissions securely.

## Key Concepts

- **GET method**: Sends data as URL query parameters. Visible in the address bar, bookmarkable, limited to roughly 2000 characters.
- **POST method**: Sends data in the request body. Not visible in the URL, no practical size limit, required for sensitive or large data.
- **Superglobal arrays**: Predefined PHP arrays (`$_GET`, `$_POST`, `$_FILES`, `$_COOKIE`, `$_SESSION`, `$_SERVER`) that are accessible from any scope.
- **Self-submitting form**: A form whose `action` points back to the same PHP file that renders it.
- **Null coalescing operator (`??`)**: Provides a default value when a key does not exist, preventing undefined index warnings.

## Real World Context

Every web application processes forms: user registration, checkout pages, search interfaces, admin panels. A single mistake in form handling can expose your application to security vulnerabilities like XSS or CSRF. The patterns you learn here — checking the request method, using `$_POST` with fallback values, and keeping display and processing in one file — are used by every major PHP framework under the hood.

## Deep Dive

### GET vs POST

Choose GET for idempotent requests (searches, filters) and POST for anything that changes state or contains sensitive data:

- **GET**: Data visible in URL, bookmarkable, limited size. Use for searches and filters.
- **POST**: Data in request body, not bookmarkable, no size limit. Use for forms, logins, and file uploads.

### Basic Form Handling

The HTML form submits to a PHP script, which reads data from the `$_POST` superglobal:

```html
<!-- contact.html -->
<form action="process.php" method="POST">
    <input type="text" name="name" required>
    <input type="email" name="email" required>
    <textarea name="message"></textarea>
    <button type="submit">Send</button>
</form>
```

```php
<?php
// process.php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $name = $_POST['name'] ?? '';
    $email = $_POST['email'] ?? '';
    $message = $_POST['message'] ?? '';
    
    // Process the data...
}
```

The `??` operator prevents warnings when a field is missing from the submission.

### Superglobal Arrays

PHP provides several superglobal arrays for different types of request data:

```php
<?php
$_GET['param'];     // URL query parameters
$_POST['field'];    // POST form data
$_REQUEST['key'];   // Combined GET + POST (avoid)
$_SERVER['key'];    // Server information
$_FILES['upload'];  // Uploaded files
$_COOKIE['name'];   // Cookie values
$_SESSION['user'];  // Session data
```

Avoid `$_REQUEST` because it merges GET and POST data, making it ambiguous which method was used.

### Self-Submitting Forms

A common pattern is a single file that both displays and processes the form:

```php
<?php
$message = '';
$name = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $name = $_POST['name'] ?? '';
    $message = "Hello, $name!";
}
?>

<form method="POST" action="">
    <input type="text" name="name" value="<?= htmlspecialchars($name) ?>">
    <button type="submit">Submit</button>
</form>

<?php if ($message): ?>
    <p><?= htmlspecialchars($message) ?></p>
<?php endif; ?>
```

Notice the use of `htmlspecialchars()` when outputting user data back to HTML. This prevents XSS attacks.

### Checking Form Submission

There are several ways to detect whether a form was submitted:

```php
<?php
// Method 1: Check request method (recommended)
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // Form was submitted
}

// Method 2: Check for specific field
if (isset($_POST['submit'])) {
    // Button was clicked
}

// Method 3: Check if any POST data exists
if (!empty($_POST)) {
    // Has POST data
}
```

Method 1 is the most reliable because it does not depend on a specific form field being present.

## Common Pitfalls

1. **Accessing `$_POST` keys without checking they exist** — Always use `$_POST['key'] ?? 'default'` or `isset($_POST['key'])`. Direct access to missing keys triggers an undefined index warning.
2. **Outputting user input without escaping** — Echoing `$_POST['name']` directly into HTML allows XSS attacks. Always wrap output in `htmlspecialchars()`.
3. **Using `$_REQUEST` instead of the specific superglobal** — `$_REQUEST` merges GET and POST data, making your code ambiguous and potentially vulnerable to parameter injection.

## Best Practices

1. **Always check the request method first** — Use `$_SERVER['REQUEST_METHOD'] === 'POST'` as your first guard before processing any form data.
2. **Trim and sanitize input early** — Apply `trim()` to string inputs immediately to remove accidental whitespace.
3. **Use the POST-Redirect-GET pattern** — After successfully processing a form submission, redirect to a GET page to prevent duplicate submissions when the user refreshes.

## Summary

- GET sends data in the URL; POST sends it in the request body. Use POST for forms.
- `$_POST`, `$_GET`, and other superglobals provide access to request data from any scope.
- Always provide default values with `??` and escape output with `htmlspecialchars()`.
- Self-submitting forms handle both display and processing in a single file.
- Check `$_SERVER['REQUEST_METHOD']` to reliably detect form submissions.

## Code Examples

**Complete form handling with validation, error messages, and safe output escaping using htmlspecialchars**

```php
<?php
// Complete form handling with validation and error display
$errors = [];
$success = false;
$formData = ['name' => '', 'email' => '', 'message' => ''];

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // Capture and trim form data
    $formData = [
        'name' => trim($_POST['name'] ?? ''),
        'email' => trim($_POST['email'] ?? ''),
        'message' => trim($_POST['message'] ?? ''),
    ];
    
    // Validate required fields
    if (empty($formData['name'])) {
        $errors['name'] = 'Name is required';
    }
    
    if (!filter_var($formData['email'], FILTER_VALIDATE_EMAIL)) {
        $errors['email'] = 'Valid email is required';
    }
    
    // Process if no errors
    if (empty($errors)) {
        // Save to database, send email, etc.
        $success = true;
    }
}
?>

<?php if ($success): ?>
    <p class="success">Thank you for your message!</p>
<?php else: ?>
    <form method="POST">
        <input name="name" value="<?= htmlspecialchars($formData['name']) ?>">
        <?php if (isset($errors['name'])): ?>
            <span class="error"><?= $errors['name'] ?></span>
        <?php endif; ?>
        
        <input name="email" value="<?= htmlspecialchars($formData['email']) ?>">
        <textarea name="message"><?= htmlspecialchars($formData['message']) ?></textarea>
        <button type="submit">Send</button>
    </form>
<?php endif; ?>
```


## Resources

- [Handling Forms](https://www.php.net/manual/en/tutorial.forms.php) — Official PHP forms tutorial

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*