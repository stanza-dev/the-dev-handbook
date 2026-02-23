---
source_course: "django-authentication"
source_lesson: "django-authentication-user-admin"
---

# Custom User Admin

## Introduction

When you customize the User model, the Django admin interface needs to be updated too. Without a custom admin class, your extra fields are invisible in the admin and the user creation form may break.

## Key Concepts

**UserAdmin**: Django's built-in admin class for the User model, providing login-safe fieldsets and password handling.

**fieldsets**: Controls which fields appear on the user detail page and in which groups.

**add_fieldsets**: Controls which fields appear on the user creation form.

**StackedInline**: Displays a related model (like Profile) inline on the user admin page.

## Real World Context

Your customer support team uses Django admin to look up users, reset passwords, and toggle account status. A well-configured UserAdmin shows the phone number, subscription tier, and last login date at a glance, so support staff can resolve issues without running database queries.

## Deep Dive

When you customize the User model, you should also customize its admin interface.

## Admin for AbstractUser Extension

```python
# admin.py
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from .models import User


@admin.register(User)
class CustomUserAdmin(UserAdmin):
    # Add custom fields to display
    list_display = (
        'username', 'email', 'first_name', 'last_name', 
        'is_staff', 'is_active', 'date_joined'
    )
    
    list_filter = ('is_staff', 'is_superuser', 'is_active', 'groups')
    
    search_fields = ('username', 'first_name', 'last_name', 'email', 'phone')
    
    ordering = ('-date_joined',)
    
    # Add custom fields to fieldsets
    fieldsets = UserAdmin.fieldsets + (
        ('Additional Info', {
            'fields': ('bio', 'birth_date', 'avatar', 'phone')
        }),
    )
    
    # Add custom fields to add form
    add_fieldsets = UserAdmin.add_fieldsets + (
        ('Additional Info', {
            'fields': ('bio', 'phone')
        }),
    )
```

## Admin for AbstractBaseUser

For email-based user models:

```python
# admin.py
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin
from django.utils.translation import gettext_lazy as _
from .models import User


@admin.register(User)
class UserAdmin(BaseUserAdmin):
    # List view
    list_display = ('email', 'first_name', 'last_name', 'is_staff', 'is_active')
    list_filter = ('is_staff', 'is_superuser', 'is_active')
    search_fields = ('email', 'first_name', 'last_name')
    ordering = ('email',)
    
    # Detail view fieldsets
    fieldsets = (
        (None, {'fields': ('email', 'password')}),
        (_('Personal info'), {'fields': ('first_name', 'last_name')}),
        (_('Permissions'), {
            'fields': ('is_active', 'is_staff', 'is_superuser', 'groups', 'user_permissions'),
        }),
        (_('Important dates'), {'fields': ('last_login', 'date_joined')}),
    )
    
    # Add user form
    add_fieldsets = (
        (None, {
            'classes': ('wide',),
            'fields': ('email', 'password1', 'password2'),
        }),
    )
    
    readonly_fields = ('date_joined', 'last_login')
```

## Profile Inline Admin

For profile model approach:

```python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin
from django.contrib.auth.models import User
from .models import Profile


class ProfileInline(admin.StackedInline):
    model = Profile
    can_delete = False
    verbose_name_plural = 'Profile'
    fk_name = 'user'


class UserAdmin(BaseUserAdmin):
    inlines = (ProfileInline,)
    list_display = ('username', 'email', 'first_name', 'last_name', 'is_staff', 'get_phone')
    list_select_related = ('profile',)
    
    def get_phone(self, obj):
        return obj.profile.phone
    get_phone.short_description = 'Phone'
    
    def get_inline_instances(self, request, obj=None):
        if not obj:
            return []
        return super().get_inline_instances(request, obj)


# Re-register User admin
admin.site.unregister(User)
admin.site.register(User, UserAdmin)
```

## Custom Actions

```python
@admin.register(User)
class UserAdmin(BaseUserAdmin):
    actions = ['activate_users', 'deactivate_users', 'send_password_reset']
    
    @admin.action(description='Activate selected users')
    def activate_users(self, request, queryset):
        count = queryset.update(is_active=True)
        self.message_user(request, f'{count} users activated.')
    
    @admin.action(description='Deactivate selected users')
    def deactivate_users(self, request, queryset):
        count = queryset.update(is_active=False)
        self.message_user(request, f'{count} users deactivated.')
    
    @admin.action(description='Send password reset email')
    def send_password_reset(self, request, queryset):
        from django.contrib.auth.forms import PasswordResetForm
        for user in queryset:
            form = PasswordResetForm({'email': user.email})
            if form.is_valid():
                form.save(request=request)
        self.message_user(request, f'Password reset sent to {queryset.count()} users.')
```

## Common Pitfalls

1. **Registering a custom User model with the default `ModelAdmin`**: The default `ModelAdmin` displays the raw hashed password field and does not use Django's password change form. Always extend `UserAdmin`.
2. **Overriding `fieldsets` completely instead of extending**: If you replace `UserAdmin.fieldsets` instead of appending to it, you lose the password change section and permission management fields.
3. **Not setting `list_select_related` for Profile inlines**: Without `list_select_related`, the admin fires a separate query for each user's profile in the list view, causing N+1 performance issues.

## Best Practices

1. **Extend `UserAdmin.fieldsets` with `+`**: Use `fieldsets = UserAdmin.fieldsets + (('Custom', {'fields': (...)}),)` to keep all built-in fields.
2. **Add `list_filter` and `search_fields`**: Make it easy for staff to find users by status, group, or email.
3. **Use custom admin actions**: Add batch operations like 'Activate users' or 'Send password reset' for common support tasks.

## Summary

- Always extend `UserAdmin`, not `ModelAdmin`, for custom user models.
- Append to `fieldsets` and `add_fieldsets` instead of replacing them.
- Use `StackedInline` or `TabularInline` for Profile models.
- Add `list_display`, `search_fields`, and `list_filter` for efficient user management.
- Custom admin actions streamline common support workflows.

## Resources

- [Django Admin Site](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/) — Complete admin site reference

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*