---
source_course: "php-essentials"
source_lesson: "php-essentials-what-is-php"
---

# What is PHP?

## Introduction
PHP is one of the most widely-used server-side programming languages on the web today. In this lesson, you will learn what PHP is, how it works under the hood, and why it remains a relevant and powerful choice for building dynamic web applications.

## Key Concepts
- **PHP (PHP: Hypertext Preprocessor)**: A recursive acronym for an open-source, server-side scripting language designed for web development.
- **Server-Side Execution**: PHP code runs on the web server, not in the user's browser. The server processes PHP and sends plain HTML to the client.
- **Embedded Language**: PHP can be embedded directly inside HTML files, making it straightforward to add dynamic behavior to web pages.

## Real World Context
PHP powers approximately 77% of all websites with a known server-side language, including WordPress, Facebook, and Wikipedia. Whether you are building a personal blog or an enterprise application, understanding PHP opens doors to a massive ecosystem of frameworks (Laravel, Symfony), CMS platforms (WordPress, Drupal), and hosting providers that support it out of the box.

## Deep Dive
PHP was originally created in 1994 by Rasmus Lerdorf as a set of Common Gateway Interface (CGI) scripts. It has since evolved into a full-featured language with modern capabilities.

Here is how a typical PHP request works:

```
1. User requests a .php page from the browser
2. Web server (Apache/Nginx) sends the request to the PHP interpreter
3. PHP processes the code, possibly querying a database
4. PHP generates HTML output
5. Server sends the generated HTML back to the user's browser
```

The key takeaway is that the browser never sees your PHP code. It only receives the final HTML output, which keeps your server logic private and secure.

PHP integrates seamlessly with databases like MySQL, PostgreSQL, and SQLite. It also has a rich standard library for file handling, string manipulation, and networking.

Let us compare PHP with other popular backend technologies:

| Feature | PHP | Node.js | Python |
|---------|-----|---------|--------|
| Learning Curve | Easy | Moderate | Easy |
| Web Focus | Native | Framework-based | Framework-based |
| Hosting | Everywhere | Specialized | Specialized |
| Package Manager | Composer | npm | pip |

PHP's tight integration with web servers and HTML makes it an excellent first server-side language to learn.

## Common Pitfalls
1. **Confusing PHP with JavaScript** — PHP runs on the server, not in the browser. If you need client-side interactivity, you still need JavaScript.
2. **Assuming PHP is outdated** — Modern PHP (8.x) includes features like JIT compilation, union types, attributes, and named arguments, making it a fully competitive language.

## Best Practices
1. **Always use the latest stable PHP version** — Newer versions bring performance improvements, security patches, and modern language features.
2. **Learn one major framework** — After mastering PHP basics, pick a framework like Laravel or Symfony to build production-grade applications efficiently.

## Summary
- PHP is a server-side scripting language designed specifically for web development.
- It processes code on the server and sends HTML to the browser.
- PHP powers the majority of the web and has a vast ecosystem of tools and frameworks.
- Modern PHP (8.x) includes powerful, up-to-date language features.

## Code Examples

**A basic PHP script showing server-side variables and output generation — notice how PHP produces HTML that gets sent to the browser**

```php
<?php
// A simple PHP script that demonstrates server-side execution
// The browser will only see the HTML output, not this PHP code

$language = "PHP";
$version = phpversion();

echo "<h1>Welcome to $language</h1>";
echo "<p>Running version: $version</p>";
echo "<p>Current server time: " . date("Y-m-d H:i:s") . "</p>";
?>
```


## Resources

- [PHP Official Documentation - Introduction](https://www.php.net/manual/en/introduction.php) — Official introduction to PHP and its capabilities

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*