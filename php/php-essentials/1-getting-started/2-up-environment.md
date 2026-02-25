---
source_course: "php-essentials"
source_lesson: "php-essentials-setting-up-environment"
---

# Setting Up Your Development Environment

## Introduction
Before writing any PHP code, you need a working development environment on your machine. This lesson walks you through the most popular setup options, from beginner-friendly all-in-one packages to professional Docker-based workflows.

## Key Concepts
- **Local Development Environment**: A set of tools (web server, PHP interpreter, database) running on your own machine for testing and development.
- **Built-in Server**: PHP ships with a lightweight development server that requires no additional software to get started.
- **All-in-One Package**: Software bundles like XAMPP or MAMP that include Apache, PHP, and MySQL in a single installer.

## Real World Context
Every professional PHP developer needs a local environment that mirrors production as closely as possible. Without one, you would have to upload code to a remote server every time you want to test a change. A properly configured local setup lets you iterate quickly, debug effectively, and test database interactions safely.

## Deep Dive
There are three main approaches to setting up PHP locally. Choose the one that fits your experience level.

### Option 1: All-in-One Packages (Recommended for Beginners)

These packages bundle everything you need in a single installation:

**XAMPP (Cross-Platform)** is the most popular choice:

```bash
# Download from https://www.apachefriends.org/
# Includes: Apache + MariaDB + PHP + Perl
```

XAMPP provides a control panel to start and stop services with one click.

**MAMP (macOS/Windows)** is another solid option:

```bash
# Download from https://www.mamp.info/
# Includes: Apache/Nginx + MySQL + PHP
```

**Laragon (Windows)** is a modern, lightweight alternative:

```bash
# Download from https://laragon.org/
# Includes: Apache/Nginx + MySQL + PHP + Node.js
```

### Option 2: Docker (Professional Setup)

For an isolated, reproducible environment that matches production, Docker is the industry standard:

```yaml
# docker-compose.yml
version: '3.8'
services:
  php:
    image: php:8.4-apache
    ports:
      - "8080:80"
    volumes:
      - ./src:/var/www/html
```

Run the container with `docker-compose up -d` and your project files in the `src/` folder are served at `http://localhost:8080`.

### Option 3: PHP Built-in Server (Quick Testing)

PHP includes a development server for quick testing with zero configuration:

```bash
# Navigate to your project folder
cd /path/to/your/project

# Start the built-in server
php -S localhost:8000
```

This is the fastest way to get started, but it is single-threaded and not suitable for production.

### Verifying Your Installation

Create a file called `info.php` and open it in your browser to confirm PHP is working:

```php
<?php
phpinfo();
?>
```

Visit `http://localhost/info.php` and you should see PHP's full configuration page.

## Common Pitfalls
1. **Leaving `info.php` on a production server** — This file exposes your entire PHP configuration, including sensitive paths and extensions. Always delete it after testing.
2. **Port conflicts** — If Apache or the built-in server fails to start, another application (like Skype or another web server) may already be using port 80 or 8000. Change the port number to resolve the conflict.

## Best Practices
1. **Use a proper code editor** — VS Code with the PHP Intelephense extension provides autocompletion, error detection, and debugging. PhpStorm is the premium choice for professional PHP development.
2. **Start with the built-in server, graduate to Docker** — The built-in server is perfect for learning. Once you work on real projects with databases, switch to Docker for consistency.

## Summary
- You need a web server, PHP interpreter, and optionally a database for local development.
- All-in-one packages (XAMPP, MAMP) are the easiest way to get started.
- The PHP built-in server (`php -S`) requires zero setup for quick testing.
- Docker provides the most professional, production-like environment.

## Code Examples

**Create this file to verify PHP is working — it displays the full PHP configuration page**

```php
<?php
// info.php - Create this file to verify your PHP installation
// Visit http://localhost/info.php in your browser
// IMPORTANT: Delete this file before deploying to production!
phpinfo();
?>
```


## Resources

- [PHP Installation Guide](https://www.php.net/manual/en/install.php) — Official PHP installation documentation for all platforms

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*