---
source_course: "laravel-eloquent-orm"
source_lesson: "laravel-eloquent-orm-one-to-one-relationships"
---

# One-to-One Relationships

A one-to-one relationship links one record to exactly one other record. For example, a User might have one Profile.

## Defining One-to-One

### hasOne (Parent Side)

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasOne;

class User extends Model
{
    /**
     * Get the user's profile.
     */
    public function profile(): HasOne
    {
        return $this->hasOne(Profile::class);
    }
}
```

### belongsTo (Child Side)

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Profile extends Model
{
    /**
     * Get the user that owns the profile.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

## Database Structure

```
users                          profiles
┌────┬──────────┐              ┌────┬─────────┬─────────────┐
│ id │ name     │              │ id │ user_id │ bio         │
├────┼──────────┤              ├────┼─────────┼─────────────┤
│ 1  │ "John"   │◄────────────│ 1  │ 1       │ "Developer" │
│ 2  │ "Jane"   │◄────────────│ 2  │ 2       │ "Designer"  │
└────┴──────────┘              └────┴─────────┴─────────────┘
```

### Migration for Profile

```php
Schema::create('profiles', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->cascadeOnDelete();
    $table->text('bio')->nullable();
    $table->string('website')->nullable();
    $table->string('location')->nullable();
    $table->timestamps();
});
```

## Using the Relationship

```php
// Access profile from user
$user = User::find(1);
$profile = $user->profile;  // Returns Profile model or null

echo $user->profile->bio;   // "Developer"

// Access user from profile
$profile = Profile::find(1);
$user = $profile->user;     // Returns User model

echo $profile->user->name;  // "John"
```

## Creating Related Records

```php
// Method 1: Create and associate
$user = User::find(1);
$profile = new Profile(['bio' => 'Developer']);
$user->profile()->save($profile);

// Method 2: Create directly
$user->profile()->create(['bio' => 'Developer']);

// Method 3: From the child side
$profile = new Profile(['bio' => 'Developer']);
$profile->user()->associate($user);
$profile->save();
```

## Customizing Keys

```php
// Custom foreign key
public function profile(): HasOne
{
    return $this->hasOne(Profile::class, 'author_id');
}

// Custom local key
public function profile(): HasOne
{
    return $this->hasOne(Profile::class, 'user_id', 'user_uuid');
}

// On the belongsTo side
public function user(): BelongsTo
{
    return $this->belongsTo(User::class, 'author_id');
}
```

## One-to-One with Default

Return a default model if none exists:

```php
public function profile(): HasOne
{
    return $this->hasOne(Profile::class)->withDefault();
}

// With default attributes
public function profile(): HasOne
{
    return $this->hasOne(Profile::class)->withDefault([
        'bio' => 'No bio provided',
    ]);
}

// Usage - never returns null
$user->profile->bio;  // "No bio provided" if no profile exists
```

## hasOne vs belongsTo

```
User (parent)           Profile (child)
┌─────────────┐         ┌─────────────────┐
│             │ hasOne  │                 │
│   User      │────────►│    Profile      │
│             │         │                 │
│             │◄────────│   user_id FK    │
│             │belongsTo│                 │
└─────────────┘         └─────────────────┘

- hasOne: "I have one of these" (User has one Profile)
- belongsTo: "I belong to this" (Profile belongs to User)
- Foreign key is on the belongsTo side (profiles.user_id)
```

## Practical Example

```php
// In a controller
public function show(User $user)
{
    // Lazy load (makes 2 queries)
    return view('users.show', [
        'user' => $user,
        'profile' => $user->profile,
    ]);

    // Better: Eager load (makes 1 query)
    $user = User::with('profile')->findOrFail($user->id);

    return view('users.show', compact('user'));
}

// In Blade
<h1>{{ $user->name }}</h1>
<p>{{ $user->profile?->bio ?? 'No bio' }}</p>
```

## Resources

- [One-to-One Relationships](https://laravel.com/docs/12.x/eloquent-relationships#one-to-one) — Official documentation on one-to-one relationships

---

> 📘 *This lesson is part of the [Laravel Eloquent ORM](https://stanza.dev/courses/laravel-eloquent-orm) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*