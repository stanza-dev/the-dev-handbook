---
source_course: "svelte-5-sveltekit"
source_lesson: "svelte-5-sveltekit-actions-basics"
---

# Form Actions: Server-Side Mutations

Form actions handle POST requests in SvelteKit. They're the preferred way to mutate data.

## Basic Form Action

```javascript
// src/routes/contact/+page.server.js
export const actions = {
  default: async ({ request }) => {
    const formData = await request.formData();
    const email = formData.get('email');
    const message = formData.get('message');
    
    await saveContactMessage({ email, message });
    
    return { success: true };
  }
};
```

```svelte
<!-- src/routes/contact/+page.svelte -->
<script>
  let { form } = $props();
</script>

{#if form?.success}
  <p class="success">Message sent!</p>
{/if}

<form method="POST">
  <input name="email" type="email" required />
  <textarea name="message" required></textarea>
  <button type="submit">Send</button>
</form>
```

## Named Actions

Have multiple actions on one page:

```javascript
// +page.server.js
export const actions = {
  login: async ({ request, cookies }) => {
    const data = await request.formData();
    const user = await authenticate(data.get('email'), data.get('password'));
    cookies.set('session', user.sessionId, { path: '/' });
    return { success: true };
  },
  
  register: async ({ request }) => {
    const data = await request.formData();
    await createUser(data.get('email'), data.get('password'));
    return { success: true };
  },
  
  logout: async ({ cookies }) => {
    cookies.delete('session', { path: '/' });
  }
};
```

```svelte
<!-- Call specific action with ?/actionName -->
<form method="POST" action="?/login">
  <input name="email" type="email" />
  <input name="password" type="password" />
  <button>Login</button>
</form>

<form method="POST" action="?/register">
  <input name="email" type="email" />
  <input name="password" type="password" />
  <button>Register</button>
</form>

<form method="POST" action="?/logout">
  <button>Logout</button>
</form>
```

## Returning Errors

```javascript
import { fail } from '@sveltejs/kit';

export const actions = {
  default: async ({ request }) => {
    const data = await request.formData();
    const email = data.get('email');
    
    if (!email.includes('@')) {
      return fail(400, { email, error: 'Invalid email address' });
    }
    
    // Success
    return { success: true };
  }
};
```

```svelte
<script>
  let { form } = $props();
</script>

{#if form?.error}
  <p class="error">{form.error}</p>
{/if}

<form method="POST">
  <input name="email" value={form?.email ?? ''} />
  <button>Submit</button>
</form>
```

📖 [Form actions documentation](https://svelte.dev/docs/kit/form-actions)

---

> 📘 *This lesson is part of the [SvelteKit 2 & Svelte 5: The Perfect Duo](https://stanza.dev/courses/svelte-5-sveltekit) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*