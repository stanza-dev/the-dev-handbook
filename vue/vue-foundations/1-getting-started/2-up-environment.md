---
source_course: "vue-foundations"
source_lesson: "vue-foundations-setting-up-environment"
---

# Setting Up Your Development Environment

Before creating Vue applications, you need to set up your development environment. Vue uses modern JavaScript tooling powered by **Vite**, a next-generation build tool that offers an incredibly fast development experience.

## Prerequisites

To develop with Vue 3.5+, you need:

- **Node.js 18.3 or higher** (we recommend the latest LTS version)
- A modern code editor (VS Code, Cursor, or WebStorm)
- A terminal application

## Step 1: Install Node.js

### On macOS

Using the official installer:
1. Download from https://nodejs.org (LTS version)
2. Run the installer

Or using Homebrew:
```bash
brew install node
```

### On Windows

1. Download from https://nodejs.org (LTS version)
2. Run the installer
3. Check "Add to PATH" during installation

### On Linux (Ubuntu/Debian)

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### Verify Installation

```bash
node --version
# v20.x.x or higher

npm --version
# 10.x.x or higher
```

## Step 2: Choose a Package Manager

Vue works with any package manager. We recommend **pnpm** for its speed and disk efficiency:

```bash
# Install pnpm globally
npm install -g pnpm

# Verify
pnpm --version
```

Alternatives:
- **npm**: Comes with Node.js (no extra installation)
- **yarn**: `npm install -g yarn`
- **bun**: Download from https://bun.sh

## Step 3: Set Up Your Code Editor

### VS Code / Cursor (Recommended)

Install these essential extensions:

1. **Vue - Official** (formerly Volar)
   - IntelliSense for Vue SFCs
   - Syntax highlighting
   - Type checking in templates

2. **TypeScript Vue Plugin**
   - Better TypeScript support in Vue files

3. **ESLint**
   - Code quality and consistency

4. **Prettier**
   - Code formatting

### WebStorm / IntelliJ IDEA

Vue support is built-in. Just ensure you have:
- Vue.js plugin enabled
- TypeScript support enabled

## Step 4: Install Vue DevTools

Vue DevTools is a browser extension for debugging Vue applications:

- **Chrome**: [Vue.js devtools](https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd)
- **Firefox**: [Vue.js devtools](https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/)
- **Edge**: [Vue.js devtools](https://microsoftedge.microsoft.com/addons/detail/vuejs-devtools/olofadcdnkkjdfgjcmjaadnlehnnihnl)

Vue DevTools allows you to:
- Inspect component hierarchy
- View and edit component state
- Track events and mutations
- Debug performance issues

## Editor Configuration

Create a `.vscode/settings.json` file for consistent settings:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "[vue]": {
    "editor.defaultFormatter": "Vue.volar"
  },
  "typescript.tsdk": "node_modules/typescript/lib"
}
```

## Troubleshooting

### "node: command not found"

Restart your terminal after installing Node.js, or add Node to your PATH manually.

### Permission errors on macOS/Linux

Don't use `sudo` with npm. Fix permissions:

```bash
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
# Add to ~/.bashrc or ~/.zshrc:
export PATH=~/.npm-global/bin:$PATH
```

### Node.js version too old

Use nvm (Node Version Manager) to manage multiple versions:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install --lts
nvm use --lts
```

## Resources

- [Vue.js Quick Start](https://vuejs.org/guide/quick-start.html) — Official guide to creating your first Vue project

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*