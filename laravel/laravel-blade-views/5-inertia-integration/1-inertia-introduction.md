---
source_course: "laravel-blade-views"
source_lesson: "laravel-blade-views-inertia-introduction"
---

# Introduction to Inertia.js

Inertia.js lets you build modern single-page applications using server-side routing and controllers. No need for an API—connect your Laravel backend directly to Vue or React.

## What is Inertia?

Inertia is the glue between your Laravel backend and JavaScript frontend:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Traditional SPA                               │
│                                                                 │
│  Frontend (React/Vue)  ←──── API ────→  Backend (Laravel)       │
│       (routing,                         (controllers,           │
│        state,                            business logic,        │
│        components)                       database)              │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    Inertia.js                                    │
│                                                                 │
│  Frontend (React/Vue)  ←── Inertia ──→  Backend (Laravel)       │
│       (components                        (controllers,          │
│        only!)                            routing,               │
│                                          business logic,        │
│                                          database)              │
└─────────────────────────────────────────────────────────────────┘
```

With Inertia:
- No API to build
- Server-side routing (like classic Laravel)
- Full SPA experience
- Use Vue or React components
- SEO friendly (server-side rendering available)

## Installation with Breeze

The easiest way to start:

```bash
composer require laravel/breeze --dev
php artisan breeze:install

# Choose Vue or React with Inertia
# [1] React with Inertia
# [2] Vue with Inertia
```

## Manual Installation

```bash
composer require inertiajs/inertia-laravel
npm install @inertiajs/vue3  # or @inertiajs/react
```

## How It Works

### Server Side (Laravel)

```php
// routes/web.php
Route::get('/users', [UserController::class, 'index']);
Route::get('/users/{user}', [UserController::class, 'show']);
Route::post('/users', [UserController::class, 'store']);
```

```php
// app/Http/Controllers/UserController.php
use Inertia\Inertia;

class UserController extends Controller
{
    public function index()
    {
        return Inertia::render('Users/Index', [
            'users' => User::all(),
        ]);
    }

    public function show(User $user)
    {
        return Inertia::render('Users/Show', [
            'user' => $user,
        ]);
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required',
            'email' => 'required|email|unique:users',
        ]);

        User::create($validated);

        return redirect()->route('users.index');
    }
}
```

### Client Side (Vue)

```vue
<!-- resources/js/Pages/Users/Index.vue -->
<script setup>
import { Link } from '@inertiajs/vue3'

defineProps({
    users: Array
})
</script>

<template>
    <h1>Users</h1>

    <Link href="/users/create">Create User</Link>

    <ul>
        <li v-for="user in users" :key="user.id">
            <Link :href="`/users/${user.id}`">{{ user.name }}</Link>
        </li>
    </ul>
</template>
```

```vue
<!-- resources/js/Pages/Users/Show.vue -->
<script setup>
defineProps({
    user: Object
})
</script>

<template>
    <h1>{{ user.name }}</h1>
    <p>{{ user.email }}</p>
</template>
```

## The Link Component

Inertia's `<Link>` replaces anchor tags for SPA navigation:

```vue
<script setup>
import { Link } from '@inertiajs/vue3'
</script>

<template>
    <!-- Basic link -->
    <Link href="/users">Users</Link>

    <!-- With method -->
    <Link href="/logout" method="post" as="button">Logout</Link>

    <!-- Preserve scroll position -->
    <Link href="/users" preserve-scroll>Users</Link>

    <!-- With data -->
    <Link href="/users" :data="{ page: 2 }">Page 2</Link>
</template>
```

## Forms

Use Inertia's form helper:

```vue
<script setup>
import { useForm } from '@inertiajs/vue3'

const form = useForm({
    name: '',
    email: '',
})

const submit = () => {
    form.post('/users')
}
</script>

<template>
    <form @submit.prevent="submit">
        <div>
            <input v-model="form.name" placeholder="Name">
            <span v-if="form.errors.name">{{ form.errors.name }}</span>
        </div>

        <div>
            <input v-model="form.email" type="email" placeholder="Email">
            <span v-if="form.errors.email">{{ form.errors.email }}</span>
        </div>

        <button type="submit" :disabled="form.processing">
            {{ form.processing ? 'Saving...' : 'Create User' }}
        </button>
    </form>
</template>
```

## Shared Data

Share data with all pages via middleware:

```php
// app/Http/Middleware/HandleInertiaRequests.php
public function share(Request $request): array
{
    return array_merge(parent::share($request), [
        'auth' => [
            'user' => $request->user(),
        ],
        'flash' => [
            'success' => $request->session()->get('success'),
            'error' => $request->session()->get('error'),
        ],
    ]);
}
```

Access in components:

```vue
<script setup>
import { usePage } from '@inertiajs/vue3'

const page = usePage()
const user = page.props.auth.user
const flash = page.props.flash
</script>

<template>
    <div v-if="flash.success" class="alert-success">
        {{ flash.success }}
    </div>

    <p v-if="user">Welcome, {{ user.name }}</p>
</template>
```

## Resources

- [Inertia.js Documentation](https://inertiajs.com/) — Official Inertia.js documentation

---

> 📘 *This lesson is part of the [Laravel Blade Templating](https://stanza.dev/courses/laravel-blade-views) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*