---
source_course: "laravel-blade-views"
source_lesson: "laravel-blade-views-tailwind-responsive"
---

# Responsive Design with Tailwind CSS

Tailwind CSS makes responsive design intuitive with mobile-first breakpoint prefixes.

## Breakpoint System

Tailwind's default breakpoints:

| Prefix | Min Width | CSS |
|--------|-----------|-----|
| (none) | 0px | `@media (min-width: 0px)` |
| `sm:` | 640px | `@media (min-width: 640px)` |
| `md:` | 768px | `@media (min-width: 768px)` |
| `lg:` | 1024px | `@media (min-width: 1024px)` |
| `xl:` | 1280px | `@media (min-width: 1280px)` |
| `2xl:` | 1536px | `@media (min-width: 1536px)` |

## Mobile-First Approach

Start with mobile styles, then add larger screen overrides:

```blade
{{-- Mobile: Stack, Desktop: Side by side --}}
<div class="flex flex-col md:flex-row">
    <div class="w-full md:w-1/3">Sidebar</div>
    <div class="w-full md:w-2/3">Content</div>
</div>
```

## Common Responsive Patterns

### Navigation Bar

```blade
<nav class="bg-white shadow">
    <div class="max-w-7xl mx-auto px-4">
        <div class="flex justify-between h-16">
            {{-- Logo --}}
            <div class="flex items-center">
                <a href="/" class="text-xl font-bold">Logo</a>
            </div>

            {{-- Desktop Navigation --}}
            <div class="hidden md:flex items-center space-x-4">
                <a href="/" class="hover:text-blue-600">Home</a>
                <a href="/about" class="hover:text-blue-600">About</a>
                <a href="/contact" class="hover:text-blue-600">Contact</a>
            </div>

            {{-- Mobile Menu Button --}}
            <div class="md:hidden flex items-center">
                <button @click="open = !open" class="p-2">
                    <svg class="w-6 h-6" fill="none" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                              d="M4 6h16M4 12h16M4 18h16" />
                    </svg>
                </button>
            </div>
        </div>
    </div>

    {{-- Mobile Menu --}}
    <div class="md:hidden" x-show="open">
        <div class="px-2 pt-2 pb-3 space-y-1">
            <a href="/" class="block px-3 py-2 hover:bg-gray-100">Home</a>
            <a href="/about" class="block px-3 py-2 hover:bg-gray-100">About</a>
            <a href="/contact" class="block px-3 py-2 hover:bg-gray-100">Contact</a>
        </div>
    </div>
</nav>
```

### Responsive Grid

```blade
{{-- 1 column mobile, 2 tablet, 3 desktop, 4 large --}}
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
    @foreach ($products as $product)
        <div class="bg-white rounded-lg shadow p-4">
            <img src="{{ $product->image }}" class="w-full h-48 object-cover rounded">
            <h3 class="mt-2 font-semibold">{{ $product->name }}</h3>
            <p class="text-gray-600">${{ $product->price }}</p>
        </div>
    @endforeach
</div>
```

### Responsive Typography

```blade
{{-- Smaller on mobile, larger on desktop --}}
<h1 class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl font-bold">
    {{ $title }}
</h1>

<p class="text-sm md:text-base lg:text-lg">
    {{ $description }}
</p>
```

### Responsive Spacing

```blade
{{-- Less padding on mobile --}}
<section class="py-8 md:py-12 lg:py-16 px-4 md:px-8">
    <div class="max-w-4xl mx-auto">
        {{ $slot }}
    </div>
</section>
```

### Hero Section

```blade
<section class="relative bg-blue-600 text-white">
    <div class="max-w-7xl mx-auto px-4 py-16 md:py-24 lg:py-32">
        <div class="md:w-2/3 lg:w-1/2">
            <h1 class="text-3xl md:text-4xl lg:text-5xl font-bold mb-4">
                Build Amazing Apps
            </h1>
            <p class="text-lg md:text-xl mb-8 opacity-90">
                Start building your next great idea today.
            </p>
            <div class="flex flex-col sm:flex-row gap-4">
                <a href="/signup" class="bg-white text-blue-600 px-6 py-3 rounded-lg
                          text-center hover:bg-gray-100">
                    Get Started
                </a>
                <a href="/learn" class="border border-white px-6 py-3 rounded-lg
                          text-center hover:bg-blue-700">
                    Learn More
                </a>
            </div>
        </div>
    </div>
</section>
```

### Card Layout

```blade
<x-layout>
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        <div class="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
            @foreach ($posts as $post)
                <article class="bg-white rounded-lg shadow-md overflow-hidden
                               hover:shadow-lg transition-shadow">
                    <img src="{{ $post->image }}"
                         class="w-full h-48 object-cover">
                    <div class="p-6">
                        <span class="text-sm text-blue-600">
                            {{ $post->category->name }}
                        </span>
                        <h2 class="mt-2 text-xl font-semibold line-clamp-2">
                            {{ $post->title }}
                        </h2>
                        <p class="mt-2 text-gray-600 line-clamp-3">
                            {{ $post->excerpt }}
                        </p>
                        <div class="mt-4 flex items-center justify-between">
                            <span class="text-sm text-gray-500">
                                {{ $post->created_at->diffForHumans() }}
                            </span>
                            <a href="{{ route('posts.show', $post) }}"
                               class="text-blue-600 hover:underline">
                                Read more →
                            </a>
                        </div>
                    </div>
                </article>
            @endforeach
        </div>
    </div>
</x-layout>
```

## Custom Breakpoints

```javascript
// tailwind.config.js
module.exports = {
    theme: {
        screens: {
            'xs': '475px',
            'sm': '640px',
            'md': '768px',
            'lg': '1024px',
            'xl': '1280px',
            '2xl': '1536px',
        },
    },
}
```

## Container Queries (Tailwind v3.4+)

```blade
{{-- Respond to container size, not viewport --}}
<div class="@container">
    <div class="@lg:flex @lg:flex-row flex-col">
        <div>Sidebar</div>
        <div>Content</div>
    </div>
</div>
```

## Resources

- [Tailwind Responsive Design](https://tailwindcss.com/docs/responsive-design) — Official Tailwind responsive design documentation

---

> 📘 *This lesson is part of the [Laravel Blade Templating](https://stanza.dev/courses/laravel-blade-views) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*