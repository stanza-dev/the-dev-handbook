---
source_course: "laravel-blade-views"
source_lesson: "laravel-blade-views-introduction-to-blade"
---

# Introduction to Blade Templates

Blade is Laravel's powerful templating engine. Unlike other PHP templating engines, Blade doesn't restrict you from using plain PHP—but it provides convenient shortcuts that make your templates cleaner and more readable.

## What is Blade?

Blade templates:
- Use the `.blade.php` extension
- Live in `resources/views/`
- Are compiled into plain PHP and cached
- Add essentially zero overhead to your application

## Creating Views

Views are stored in `resources/views/`:

```
resources/views/
├── welcome.blade.php
├── layouts/
│   └── app.blade.php
├── posts/
│   ├── index.blade.php
│   ├── show.blade.php
│   └── create.blade.php
└── components/
    └── alert.blade.php
```

## Returning Views

From routes:

```php
// Simple view
Route::get('/', function () {
    return view('welcome');
});

// Nested view (posts/index.blade.php)
Route::get('/posts', function () {
    return view('posts.index');
});
```

From controllers:

```php
class PostController extends Controller
{
    public function index()
    {
        $posts = Post::all();

        return view('posts.index', ['posts' => $posts]);
    }

    // Alternative syntax
    public function show(Post $post)
    {
        return view('posts.show', compact('post'));
    }

    // Using with()
    public function create()
    {
        return view('posts.create')
            ->with('categories', Category::all());
    }
}
```

## Displaying Data

Blade's double curly braces echo data with automatic HTML escaping:

```blade
<!-- Escaped output (safe from XSS) -->
<h1>{{ $title }}</h1>
<p>{{ $user->name }}</p>
<p>{{ $user->bio ?? 'No bio provided' }}</p>

<!-- With HTML entities escaped -->
{{ '<script>alert("XSS")</script>' }}
<!-- Output: &lt;script&gt;alert("XSS")&lt;/script&gt; -->
```

### Unescaped Data

For trusted HTML content (use carefully!):

```blade
<!-- Unescaped - ONLY for trusted content -->
{!! $post->html_content !!}

<!-- WARNING: This is vulnerable to XSS if data isn't sanitized -->
```

## PHP in Blade

You can use raw PHP when needed:

```blade
@php
    $currentYear = date('Y');
    $greeting = $hour < 12 ? 'Good morning' : 'Good afternoon';
@endphp

<footer>&copy; {{ $currentYear }}</footer>
```

## Comments

```blade
{{-- This comment will NOT appear in the HTML output --}}

<!-- This HTML comment WILL appear in the output -->
```

## Verbatim (Escaping Blade)

For JavaScript frameworks that use `{{ }}`:

```blade
<!-- Single escape -->
@{{ vueVariable }}

<!-- Block escape -->
@verbatim
    <div id="app">
        {{ message }}
        {{ user.name }}
    </div>
@endverbatim
```

## Checking View Existence

```php
if (View::exists('posts.show')) {
    return view('posts.show', $data);
}

// Or return first existing view
return view()->first(['custom.posts', 'posts.show'], $data);
```

## Sharing Data with All Views

```php
// In AppServiceProvider boot()
View::share('appName', config('app.name'));
View::share('currentUser', auth()->user());

// Now available in ALL views
<title>{{ $appName }}</title>
```

## View Composers

Automatically bind data to specific views:

```php
// In AppServiceProvider boot()
View::composer('layouts.app', function ($view) {
    $view->with('notifications', auth()->user()?->unreadNotifications);
});

// For multiple views
View::composer(['posts.*', 'pages.*'], function ($view) {
    $view->with('categories', Category::all());
});

// Using a class
View::composer('profile', ProfileComposer::class);
```

```php
// app/View/Composers/ProfileComposer.php
class ProfileComposer
{
    public function compose(View $view): void
    {
        $view->with('stats', [
            'posts' => auth()->user()->posts()->count(),
            'followers' => auth()->user()->followers()->count(),
        ]);
    }
}
```

## Resources

- [Blade Templates](https://laravel.com/docs/12.x/blade) — Official Blade documentation

---

> 📘 *This lesson is part of the [Laravel Blade Templating](https://stanza.dev/courses/laravel-blade-views) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*