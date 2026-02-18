---
source_course: "laravel-blade-views"
source_lesson: "laravel-blade-views-livewire-forms-validation"
---

# Livewire Forms and Validation

Build powerful forms with real-time validation using Livewire.

## Basic Form

```php
<?php

namespace App\Livewire;

use App\Models\Post;
use Livewire\Component;

class CreatePost extends Component
{
    public string $title = '';
    public string $body = '';

    protected array $rules = [
        'title' => 'required|min:3|max:255',
        'body' => 'required|min:10',
    ];

    public function save(): void
    {
        $validated = $this->validate();

        Post::create($validated);

        $this->reset();  // Clear form
        session()->flash('success', 'Post created!');
    }

    public function render()
    {
        return view('livewire.create-post');
    }
}
```

```blade
<div>
    @if (session('success'))
        <div class="bg-green-100 p-4 mb-4">{{ session('success') }}</div>
    @endif

    <form wire:submit="save">
        <div class="mb-4">
            <label>Title</label>
            <input type="text" wire:model="title" class="w-full border p-2">
            @error('title')
                <span class="text-red-500 text-sm">{{ $message }}</span>
            @enderror
        </div>

        <div class="mb-4">
            <label>Body</label>
            <textarea wire:model="body" rows="5" class="w-full border p-2"></textarea>
            @error('body')
                <span class="text-red-500 text-sm">{{ $message }}</span>
            @enderror
        </div>

        <button type="submit" class="bg-blue-500 text-white px-4 py-2">
            Create Post
        </button>
    </form>
</div>
```

## Real-Time Validation

Validate as the user types:

```php
class CreatePost extends Component
{
    public string $title = '';
    public string $body = '';

    protected array $rules = [
        'title' => 'required|min:3|max:255',
        'body' => 'required|min:10',
    ];

    // Called when any property is updated
    public function updated(string $propertyName): void
    {
        $this->validateOnly($propertyName);
    }

    // Or for specific properties
    public function updatedTitle(): void
    {
        $this->validateOnly('title');
    }

    public function save(): void
    {
        $this->validate();
        // ...
    }
}
```

```blade
{{-- Use wire:model.live for real-time validation --}}
<input type="text" wire:model.live.debounce.500ms="title">
```

## Form Objects

For complex forms, use Livewire Form Objects:

```bash
php artisan make:livewire-form PostForm
```

```php
<?php

namespace App\Livewire\Forms;

use Livewire\Form;
use App\Models\Post;

class PostForm extends Form
{
    public ?Post $post = null;

    public string $title = '';
    public string $body = '';
    public bool $published = false;

    protected function rules(): array
    {
        return [
            'title' => 'required|min:3|max:255',
            'body' => 'required|min:10',
            'published' => 'boolean',
        ];
    }

    public function setPost(Post $post): void
    {
        $this->post = $post;
        $this->title = $post->title;
        $this->body = $post->body;
        $this->published = $post->published;
    }

    public function store(): Post
    {
        $this->validate();

        return Post::create($this->only(['title', 'body', 'published']));
    }

    public function update(): void
    {
        $this->validate();

        $this->post->update($this->only(['title', 'body', 'published']));
    }
}
```

Using the form:

```php
class ManagePost extends Component
{
    public PostForm $form;

    public function mount(?Post $post = null): void
    {
        if ($post) {
            $this->form->setPost($post);
        }
    }

    public function save(): void
    {
        if ($this->form->post) {
            $this->form->update();
        } else {
            $this->form->store();
        }

        $this->redirect(route('posts.index'));
    }

    public function render()
    {
        return view('livewire.manage-post');
    }
}
```

```blade
<form wire:submit="save">
    <input type="text" wire:model="form.title">
    @error('form.title') <span>{{ $message }}</span> @enderror

    <textarea wire:model="form.body"></textarea>
    @error('form.body') <span>{{ $message }}</span> @enderror

    <label>
        <input type="checkbox" wire:model="form.published">
        Published
    </label>

    <button type="submit">Save</button>
</form>
```

## File Uploads

```php
use Livewire\WithFileUploads;

class UploadPhoto extends Component
{
    use WithFileUploads;

    public $photo;

    protected array $rules = [
        'photo' => 'required|image|max:1024',  // 1MB max
    ];

    public function save(): void
    {
        $this->validate();

        $path = $this->photo->store('photos', 'public');

        auth()->user()->update(['avatar' => $path]);

        session()->flash('success', 'Photo uploaded!');
    }

    public function render()
    {
        return view('livewire.upload-photo');
    }
}
```

```blade
<form wire:submit="save">
    <input type="file" wire:model="photo">

    {{-- Preview --}}
    @if ($photo)
        <img src="{{ $photo->temporaryUrl() }}" class="w-32 h-32">
    @endif

    @error('photo') <span>{{ $message }}</span> @enderror

    <button type="submit" wire:loading.attr="disabled">
        <span wire:loading.remove>Upload</span>
        <span wire:loading>Uploading...</span>
    </button>
</form>
```

## Custom Validation Messages

```php
protected array $messages = [
    'title.required' => 'Please enter a title.',
    'title.min' => 'The title must be at least :min characters.',
    'body.required' => 'Don\'t forget to add content!',
];

protected array $validationAttributes = [
    'title' => 'post title',
    'body' => 'post content',
];
```

## Resources

- [Livewire Forms](https://livewire.laravel.com/docs/forms) — Livewire forms and validation

---

> 📘 *This lesson is part of the [Laravel Blade Templating](https://stanza.dev/courses/laravel-blade-views) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*