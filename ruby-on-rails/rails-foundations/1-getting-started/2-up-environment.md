---
source_course: "rails-foundations"
source_lesson: "rails-foundations-setting-up-environment"
---

# Setting Up Your Rails Environment

## Introduction

Before creating Rails applications, you need to install Ruby and the Rails gem. A properly configured development environment is the foundation for productive Rails development.

## Key Concepts

- **Ruby**: The programming language Rails is built on. Rails 8.1 requires Ruby 3.2 or newer.
- **rbenv**: A lightweight Ruby version manager that lets you install and switch between Ruby versions.
- **Gems**: Ruby packages/libraries. Rails itself is a gem, installed via `gem install rails`.
- **Bundler**: A dependency manager for Ruby that ensures consistent gem versions across environments.
- **SQLite3**: The default database for Rails development, requiring zero configuration.

## Real World Context

Professional Rails developers manage multiple Ruby versions across projects. Using a version manager like rbenv ensures you can work on different projects without conflicts. Getting the environment right once saves hours of debugging later.

## Deep Dive

### Step 1: Install Ruby

#### On macOS

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

#### On Ubuntu/Debian

```bash
sudo apt-get update
sudo apt-get install git curl libssl-dev libreadline-dev zlib1g-dev
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
source ~/.bashrc
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
rbenv install 3.3.0
rbenv global 3.3.0
```

### Step 2: Install Rails

```bash
gem install rails
rails --version
# => Rails 8.1.x
```

### Step 3: Install SQLite3

```bash
# macOS
brew install sqlite3

# Ubuntu/Debian
sudo apt-get install sqlite3 libsqlite3-dev
```

### Recommended Code Editor

We recommend **VS Code** with these extensions:
- **Ruby LSP** - Language support for Ruby
- **Rails** - Rails-specific snippets and navigation
- **ERB Formatter/Beautify** - Format ERB templates

## Common Pitfalls

- **Using `sudo gem install`**: Never use sudo. Instead, ensure rbenv is properly configured so gems install in your home directory.
- **Wrong Ruby version**: Rails 8.1 requires Ruby 3.2+. Check with `ruby --version` before installing Rails.
- **Missing build tools**: On Linux, you need development headers (libssl-dev, etc.) before compiling Ruby.

## Best Practices

- Use a version manager (rbenv or asdf) rather than system Ruby.
- Pin your Ruby version per project with a `.ruby-version` file.
- Keep your gems and bundler up to date with `gem update --system`.

## Summary

- Rails requires Ruby 3.2+, SQLite3, and optionally Node.js.
- Use rbenv to manage Ruby versions and avoid permission issues.
- Install Rails with `gem install rails` and verify with `rails --version`.
- A proper development setup prevents hours of debugging later.

## Code Examples

**Essential commands for setting up a Rails development environment with rbenv and verifying the installation.**

```ruby
# Install Ruby with rbenv
# brew install rbenv ruby-build
# rbenv install 3.3.0
# rbenv global 3.3.0

# Install Rails
# gem install rails

# Verify installation
# ruby --version   => ruby 3.3.0
# rails --version  => Rails 8.1.x
```


## Resources

- [Install Ruby on Rails](https://guides.rubyonrails.org/install_ruby_on_rails.html) — Official Rails installation guide for all platforms

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*