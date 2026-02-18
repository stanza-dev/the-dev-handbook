---
source_course: "php-essentials"
source_lesson: "php-essentials-handling-forms"
---

# Handling Form Submissions

Forms are the primary way users interact with PHP applications. Understanding how to handle form data securely is essential.

## GET vs POST Methods

### GET Method
- Data visible in URL
- Bookmarkable
- Limited data size (~2000 chars)
- Use for: searches, filters, non-sensitive data

### POST Method
- Data in request body (not visible in URL)
- Not bookmarkable
- No size limit
- Use for: forms, sensitive data, file uploads

## Basic Form Handling

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

## Superglobal Arrays

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

## Self-Submitting Forms

```php
<?php
// Single file handles both display and processing
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

## Checking Form Submission

```php
<?php
// Method 1: Check request method
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

## Code Examples

**Complete form with validation and error handling**

```php
<?php
// Complete form handling example
$errors = [];
$success = false;
$formData = ['name' => '', 'email' => '', 'message' => ''];

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // Capture form data
    $formData = [
        'name' => trim($_POST['name'] ?? ''),
        'email' => trim($_POST['email'] ?? ''),
        'message' => trim($_POST['message'] ?? ''),
    ];
    
    // Basic validation
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