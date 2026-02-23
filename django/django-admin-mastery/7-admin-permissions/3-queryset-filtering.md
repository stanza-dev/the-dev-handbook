---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-admin-queryset-filtering"
---

# QuerySet Filtering for Permissions

## Introduction

Use get_queryset() to implement object-level permissions by filtering what users can see.

## Key Concepts

**get_queryset()**: Controls which objects appear in the list.

**Object-level permissions**: Different users see different objects.

**save_model()**: Auto-assign ownership on creation.

## Real World Context

A multi-branch retail company uses the admin for inventory management. Each store manager should only see and edit products in their branch. By filtering get_queryset() to match the user's branch and auto-assigning the branch in save_model(), the admin enforces data isolation without any custom middleware or third-party packages.

## Deep Dive

### Filter by Owner

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    def get_queryset(self, request):
        qs = super().get_queryset(request)
        if request.user.is_superuser:
            return qs
        return qs.filter(author=request.user)

    def save_model(self, request, obj, form, change):
        if not change:  # New object
            obj.author = request.user
        super().save_model(request, obj, form, change)
```

### Filter by Group

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    def get_queryset(self, request):
        qs = super().get_queryset(request)
        if request.user.groups.filter(name='Editors').exists():
            return qs  # Editors see all
        return qs.filter(author=request.user)

    def get_readonly_fields(self, request, obj=None):
        if not request.user.groups.filter(name='Editors').exists():
            return ['status', 'pub_date']  # Non-editors can't change these
        return []
```

### Limit Choices in Foreign Keys

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    def formfield_for_foreignkey(self, db_field, request, **kwargs):
        if db_field.name == 'author':
            if not request.user.is_superuser:
                kwargs['queryset'] = User.objects.filter(
                    groups__name='Authors'
                )
        return super().formfield_for_foreignkey(db_field, request, **kwargs)

    def formfield_for_manytomany(self, db_field, request, **kwargs):
        if db_field.name == 'categories':
            kwargs['queryset'] = Category.objects.filter(is_active=True)
        return super().formfield_for_manytomany(db_field, request, **kwargs)
```

## Common Pitfalls

1. **Not calling super().get_queryset()**: Starting with `self.model.objects.all()` instead of `super().get_queryset(request)` bypasses any default filtering, annotations, or select_related that Django or other mixins apply.

2. **Forgetting formfield_for_foreignkey()**: Even if get_queryset() hides other branches' products, a dropdown for 'transfer_to_branch' still shows all branches. Use formfield_for_foreignkey() to filter dropdown choices consistently.

## Best Practices

1. **Always call super()**: Build on the default queryset.
2. **Auto-assign author**: Use save_model() for ownership.

## Summary

get_queryset() controls object visibility per user. Combine with save_model() for automatic ownership. Use formfield_for_foreignkey() to limit dropdown choices.

## Resources

- [ModelAdmin.get_queryset](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.get_queryset) — Filtering admin querysets

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*