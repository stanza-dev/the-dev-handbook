---
source_course: "php-essentials"
source_lesson: "php-essentials-setting-up-environment"
---

# Setting Up Your Development Environment

Before writing PHP code, you need a local development environment. This includes a web server, PHP interpreter, and optionally a database.

## Option 1: All-in-One Packages (Recommended for Beginners)

These packages bundle everything you need:

### XAMPP (Cross-Platform)
```bash
# Download from https://www.apachefriends.org/
# Includes: Apache + MariaDB + PHP + Perl
```

### MAMP (macOS/Windows)
```bash
# Download from https://www.mamp.info/
# Includes: Apache/Nginx + MySQL + PHP
```

### Laragon (Windows - Modern Choice)
```bash
# Download from https://laragon.org/
# Includes: Apache/Nginx + MySQL + PHP + Node.js
```

## Option 2: Docker (Professional Setup)

For a more professional, isolated environment:

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

Run with: `docker-compose up -d`

## Option 3: PHP Built-in Server (Quick Testing)

PHP includes a development server for quick testing:

```bash
# Navigate to your project folder
cd /path/to/your/project

# Start the built-in server
php -S localhost:8000
```

## Verifying Your Installation

Create a file called `info.php`:

```php
<?php
phpinfo();
?>
```

Visit `http://localhost/info.php` in your browser. You should see PHP's configuration page.

## Recommended Code Editor

- **VS Code** with PHP Intelephense extension
- **PhpStorm** (professional, paid)
- **Sublime Text** with PHP packages

## Code Examples

**Create this file to verify PHP is working**

```php
<?php
// info.php - Test your PHP installation
phpinfo();
?>
```


## Resources

- [PHP Installation Guide](https://www.php.net/manual/en/install.php) — Official PHP installation documentation for all platforms

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*