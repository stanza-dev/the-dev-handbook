---
source_course: "laravel-foundations"
source_lesson: "laravel-foundations-setting-up-environment"
---

# Setting Up Your Laravel Environment

Before creating Laravel applications, you need to install PHP, Composer, and the Laravel installer. Let's get everything ready for development.

## Prerequisites

Laravel 12 requires:
- **PHP 8.2 or higher** (we recommend PHP 8.4)
- **Composer** (PHP package manager)
- **Node.js and NPM** (for frontend assets)
- A code editor (VS Code, PhpStorm, or Cursor)

## Step 1: Install PHP and Composer

The easiest way to install PHP, Composer, and the Laravel installer is using the official installation scripts.

### On macOS

```bash
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.4)"
```

This installs:
- PHP 8.4
- Composer
- Laravel installer

Restart your terminal after installation.

### On Windows (PowerShell as Administrator)

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.4'))
```

### On Linux (Ubuntu/Debian)

```bash
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.4)"
```

### Verify Installation

```bash
# Check PHP version
php --version
# PHP 8.4.x

# Check Composer
composer --version
# Composer version 2.x.x

# Check Laravel installer
laravel --version
# Laravel Installer x.x.x
```

## Step 2: Install Node.js

Laravel uses Vite for frontend asset compilation. You need Node.js:

### Using the Official Installer

Download from https://nodejs.org (LTS version recommended)

### Using nvm (Node Version Manager) - Recommended

```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Install Node.js
nvm install --lts

# Verify
node --version
npm --version
```

### Alternative: Using Bun

Laravel also supports Bun as a faster alternative:

```bash
curl -fsSL https://bun.sh/install | bash
```

## Step 3: Choose a Code Editor

We recommend these editors for Laravel development:

### VS Code / Cursor

Install these extensions:
- **PHP Intelephense** - Smart PHP support
- **Laravel Extension Pack** - Blade, Artisan, and more
- **Laravel Blade Snippets** - Blade template support
- **DotENV** - Environment file syntax

### PhpStorm

PhpStorm has excellent built-in Laravel support including:
- Blade template support
- Eloquent model autocompletion
- Route and view navigation
- Artisan command integration

## Alternative: Laravel Herd

Laravel Herd is an all-in-one development environment that includes PHP, Nginx, and more.

### On macOS

1. Download from https://herd.laravel.com
2. The installer sets up everything automatically
3. Applications in `~/Herd` are served on `.test` domains

```bash
cd ~/Herd
laravel new my-app
cd my-app
herd open  # Opens http://my-app.test in browser
```

### On Windows

1. Download Herd for Windows
2. Applications go in `%USERPROFILE%\Herd`
3. Same `.test` domain feature

## Troubleshooting

### "php: command not found"

Restart your terminal or add PHP to your PATH.

### "Permission denied" on macOS/Linux

Don't use `sudo` with Composer. Fix permissions instead:

```bash
sudo chown -R $USER ~/.composer
```

### Composer memory errors

Increase memory limit:

```bash
COMPOSER_MEMORY_LIMIT=-1 composer install
```

## Resources

- [Laravel Installation Guide](https://laravel.com/docs/12.x/installation) — Official guide to installing Laravel on all platforms

---

> 📘 *This lesson is part of the [Laravel Foundations](https://stanza.dev/courses/laravel-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*