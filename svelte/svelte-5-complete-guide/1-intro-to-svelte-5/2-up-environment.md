---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-setting-up-environment"
---

# Getting Started with Svelte 5

Before you can build Svelte applications, you need to set up your development environment. The good news is that Svelte's tooling is excellent and easy to configure.

## Prerequisites

You'll need **Node.js** version 18 or higher installed on your computer. You can check your version by running:

```bash
node --version
```

## Creating a New Svelte Project

The official way to create a Svelte project is using the `sv` CLI (Svelte's official command-line tool):

```bash
npx sv create my-app
cd my-app
npm install
npm run dev
```

This creates a **SvelteKit** project — the official framework for building Svelte applications. SvelteKit handles routing, server-side rendering, and much more.

During setup, you'll be asked a few questions:
- **Template**: Choose "SvelteKit minimal" for learning
- **TypeScript**: Recommended, but optional
- **Add-ons**: You can skip these for now

## Project Structure

After creation, your project looks like this:

```
my-app/
├── src/
│   ├── routes/          # Your pages live here
│   │   └── +page.svelte # The home page
│   ├── lib/             # Reusable code
│   └── app.html         # HTML template
├── static/              # Static assets
├── package.json
├── svelte.config.js     # Svelte configuration
└── vite.config.js       # Build tool configuration
```

## The Development Server

When you run `npm run dev`, Vite starts a development server with:

- **Hot Module Replacement (HMR)**: Changes appear instantly without page refresh
- **Fast Compilation**: Svelte compiles in milliseconds
- **Error Overlay**: Errors appear directly in the browser

Open `http://localhost:5173` to see your app running!

## Your First Edit

Open `src/routes/+page.svelte` and change the content:

```svelte
<h1>Hello Svelte 5!</h1>
<p>My first Svelte app is running.</p>
```

Save the file and watch your browser update instantly. That's Svelte's development experience!

## Resources

- [Getting Started with Svelte](https://svelte.dev/docs/svelte/getting-started) — Official guide for creating your first Svelte project.

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*