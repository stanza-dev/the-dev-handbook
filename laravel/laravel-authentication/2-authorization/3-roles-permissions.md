---
source_course: "laravel-authentication"
source_lesson: "laravel-authentication-roles-permissions"
---

# Implementing Roles and Permissions

Build a flexible role-based access control (RBAC) system for your Laravel application.

## Basic Role System

### Database Structure

```php
// Migration: create_roles_table
Schema::create('roles', function (Blueprint $table) {
    $table->id();
    $table->string('name')->unique();       // 'admin', 'editor'
    $table->string('display_name');         // 'Administrator', 'Editor'
    $table->text('description')->nullable();
    $table->timestamps();
});

// Migration: create_role_user_table (pivot)
Schema::create('role_user', function (Blueprint $table) {
    $table->foreignId('role_id')->constrained()->cascadeOnDelete();
    $table->foreignId('user_id')->constrained()->cascadeOnDelete();
    $table->primary(['role_id', 'user_id']);
});
```

### Models

```php
// app/Models/Role.php
class Role extends Model
{
    protected $fillable = ['name', 'display_name', 'description'];

    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class);
    }
}

// app/Models/User.php
class User extends Authenticatable
{
    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class);
    }

    public function hasRole(string $role): bool
    {
        return $this->roles()->where('name', $role)->exists();
    }

    public function hasAnyRole(array $roles): bool
    {
        return $this->roles()->whereIn('name', $roles)->exists();
    }

    public function assignRole(string $role): void
    {
        $role = Role::where('name', $role)->firstOrFail();
        $this->roles()->syncWithoutDetaching($role);
    }

    public function removeRole(string $role): void
    {
        $role = Role::where('name', $role)->firstOrFail();
        $this->roles()->detach($role);
    }
}
```

### Using Roles in Gates

```php
// AppServiceProvider
Gate::before(function (User $user, string $ability) {
    if ($user->hasRole('super-admin')) {
        return true;
    }
});

Gate::define('manage-users', function (User $user) {
    return $user->hasAnyRole(['admin', 'super-admin']);
});

Gate::define('edit-articles', function (User $user) {
    return $user->hasAnyRole(['editor', 'admin', 'super-admin']);
});
```

## Advanced: Permissions System

### Database Structure

```php
// Permissions table
Schema::create('permissions', function (Blueprint $table) {
    $table->id();
    $table->string('name')->unique();  // 'posts.create', 'users.delete'
    $table->string('display_name');
    $table->timestamps();
});

// Role-Permission pivot
Schema::create('permission_role', function (Blueprint $table) {
    $table->foreignId('permission_id')->constrained()->cascadeOnDelete();
    $table->foreignId('role_id')->constrained()->cascadeOnDelete();
    $table->primary(['permission_id', 'role_id']);
});
```

### Permission Model

```php
class Permission extends Model
{
    protected $fillable = ['name', 'display_name'];

    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class);
    }
}

// Update Role model
class Role extends Model
{
    public function permissions(): BelongsToMany
    {
        return $this->belongsToMany(Permission::class);
    }

    public function hasPermission(string $permission): bool
    {
        return $this->permissions()->where('name', $permission)->exists();
    }
}
```

### User Permission Methods

```php
class User extends Authenticatable
{
    public function permissions(): Collection
    {
        return $this->roles
            ->flatMap(fn ($role) => $role->permissions)
            ->unique('id');
    }

    public function hasPermission(string $permission): bool
    {
        return $this->permissions()->contains('name', $permission);
    }
}
```

### Gate Integration

```php
// AppServiceProvider
public function boot(): void
{
    // Define gates from permissions
    Permission::all()->each(function ($permission) {
        Gate::define($permission->name, function (User $user) use ($permission) {
            return $user->hasPermission($permission->name);
        });
    });

    // Super admin bypass
    Gate::before(function (User $user, string $ability) {
        if ($user->hasRole('super-admin')) {
            return true;
        }
    });
}
```

## Role Middleware

```php
// app/Http/Middleware/CheckRole.php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class CheckRole
{
    public function handle(Request $request, Closure $next, string ...$roles)
    {
        if (! $request->user() || ! $request->user()->hasAnyRole($roles)) {
            abort(403, 'Unauthorized.');
        }

        return $next($request);
    }
}
```

Register in `bootstrap/app.php`:

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'role' => \App\Http\Middleware\CheckRole::class,
    ]);
})
```

Usage:

```php
// Single role
Route::get('/admin', AdminController::class)->middleware('role:admin');

// Multiple roles (OR)
Route::get('/dashboard', DashboardController::class)
    ->middleware('role:admin,editor');
```

## Blade Directives

```php
// AppServiceProvider
Blade::if('role', function (string $role) {
    return auth()->check() && auth()->user()->hasRole($role);
});

Blade::if('permission', function (string $permission) {
    return auth()->check() && auth()->user()->hasPermission($permission);
});
```

```blade
@role('admin')
    <a href="/admin">Admin Panel</a>
@endrole

@permission('posts.create')
    <a href="/posts/create">New Post</a>
@endpermission
```

## Caching Permissions

For performance, cache permissions:

```php
class User extends Authenticatable
{
    public function permissions(): Collection
    {
        return Cache::remember(
            "user.{$this->id}.permissions",
            now()->addMinutes(60),
            fn () => $this->roles
                ->flatMap(fn ($role) => $role->permissions)
                ->unique('id')
        );
    }

    // Clear cache when roles change
    public function clearPermissionCache(): void
    {
        Cache::forget("user.{$this->id}.permissions");
    }
}
```

## Resources

- [Authorization](https://laravel.com/docs/12.x/authorization) — Complete authorization documentation

---

> 📘 *This lesson is part of the [Laravel Authentication & Authorization](https://stanza.dev/courses/laravel-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*