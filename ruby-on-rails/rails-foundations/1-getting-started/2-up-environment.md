---
source_course: "rails-foundations"
source_lesson: "rails-foundations-setting-up-environment"
---

# Setting Up Your Rails Environment

Before creating Rails applications, you need to install Ruby and the Rails gem. Let's get your development environment ready.

## Prerequisites

Rails requires:
- **Ruby** 3.1.0 or newer (Rails 8 requires Ruby 3.2+)
- **SQLite3** (default database for development)
- **Node.js** (for JavaScript bundling, optional with import maps)

## Step 1: Install Ruby

### On macOS

Using Homebrew and rbenv (recommended):

```bash
# Install rbenv and ruby-build
brew install rbenv ruby-build

# Add rbenv to your shell
echo 'eval "$(rbenv init -)"' >> ~/.zshrc
source ~/.zshrc

# Install Ruby
rbenv install 3.3.0
rbenv global 3.3.0

# Verify installation
ruby --version
```

### On Ubuntu/Debian

```bash
# Install dependencies
sudo apt-get update
sudo apt-get install git curl libssl-dev libreadline-dev zlib1g-dev

# Install rbenv
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
source ~/.bashrc

# Install ruby-build plugin
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build

# Install Ruby
rbenv install 3.3.0
rbenv global 3.3.0
```

### On Windows

Use **RubyInstaller** from https://rubyinstaller.org/:
1. Download Ruby+Devkit version
2. Run the installer
3. Select "Add Ruby executables to your PATH"

## Step 2: Install Rails

Once Ruby is installed:

```bash
# Install the Rails gem
gem install rails

# Verify installation
rails --version
# => Rails 8.0.x
```

## Step 3: Install SQLite3

### macOS
```bash
brew install sqlite3
```

### Ubuntu/Debian
```bash
sudo apt-get install sqlite3 libsqlite3-dev
```

### Verify SQLite
```bash
sqlite3 --version
```

## Recommended Code Editor

We recommend **VS Code** with these extensions:
- **Ruby LSP** - Language support for Ruby
- **Rails** - Rails-specific snippets and navigation
- **ERB Formatter/Beautify** - Format ERB templates

## Troubleshooting Common Issues

### "Permission denied" errors
Never use `sudo gem install`. Instead, ensure rbenv is properly configured.

### "Could not find gem" errors
```bash
gem update --system
bundle update
```

### Database connection issues
Ensure SQLite3 is installed and the `sqlite3` gem compiles correctly.

## Resources

- [Install Ruby on Rails](https://guides.rubyonrails.org/install_ruby_on_rails.html) — Official Rails installation guide for all platforms

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*