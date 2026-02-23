---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-list-filters"
---

# List Filters

## Introduction

Filters let admins narrow down the change list to find specific objects quickly. Django provides built-in field filters and a SimpleListFilter base class for creating custom filtering logic.

## Key Concepts

**list_filter**: Attribute listing fields or filter classes for the sidebar.

**SimpleListFilter**: Base class for custom filters with lookups() and queryset() methods.

**RelatedOnlyFieldListFilter**: Shows only related objects that have associations, instead of all objects.

**parameter_name**: The URL query parameter used by a SimpleListFilter.

## Real World Context

A hospital administration system has thousands of patient records. Front desk staff need to filter by appointment date, department, and insurance status. A custom DateRangeFilter for 'Today', 'This Week', 'This Month' combined with built-in filters for department and a SimpleListFilter for insurance status turns the admin list into a practical patient lookup tool.

## Deep Dive

Filters help admins find specific objects quickly. Django provides built-in filters and lets you create custom ones.

### Built-in Filters

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_filter = [
        'status',              # ChoiceField
        'is_featured',         # BooleanField
        'pub_date',            # DateField (with ranges)
        'author',              # ForeignKey
        'tags',                # ManyToMany
        ('author', admin.RelatedOnlyFieldListFilter),  # Only show authors with articles
    ]
```

### SimpleListFilter

```python
from django.contrib.admin import SimpleListFilter


class PublishedYearFilter(SimpleListFilter):
    title = 'published year'
    parameter_name = 'pub_year'
    
    def lookups(self, request, model_admin):
        """Return list of (value, label) tuples."""
        years = Article.objects.dates('pub_date', 'year')
        return [(str(y.year), str(y.year)) for y in years]
    
    def queryset(self, request, queryset):
        """Filter queryset based on selected value."""
        if self.value():
            return queryset.filter(pub_date__year=self.value())
        return queryset


class WordCountFilter(SimpleListFilter):
    title = 'word count'
    parameter_name = 'words'
    
    def lookups(self, request, model_admin):
        return [
            ('short', 'Short (< 500 words)'),
            ('medium', 'Medium (500-2000 words)'),
            ('long', 'Long (> 2000 words)'),
        ]
    
    def queryset(self, request, queryset):
        if self.value() == 'short':
            return queryset.annotate(
                word_count=Length('body') - Length(Replace('body', Value(' '), Value(''))) + 1
            ).filter(word_count__lt=500)
        # ... etc


@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_filter = [PublishedYearFilter, WordCountFilter, 'status']
```

### Filter with Request Context

```python
class MyArticlesFilter(SimpleListFilter):
    """Filter to show only current user's articles."""
    title = 'ownership'
    parameter_name = 'mine'
    
    def lookups(self, request, model_admin):
        if request.user.is_superuser:
            return [
                ('mine', 'My articles'),
                ('others', 'Other users'),
            ]
        return [('mine', 'My articles')]
    
    def queryset(self, request, queryset):
        if self.value() == 'mine':
            return queryset.filter(author=request.user)
        if self.value() == 'others':
            return queryset.exclude(author=request.user)
        return queryset
```

### Date Range Filter

```python
class DateRangeFilter(SimpleListFilter):
    title = 'date range'
    parameter_name = 'date_range'
    
    def lookups(self, request, model_admin):
        return [
            ('today', 'Today'),
            ('week', 'This week'),
            ('month', 'This month'),
            ('quarter', 'This quarter'),
            ('year', 'This year'),
        ]
    
    def queryset(self, request, queryset):
        from datetime import date, timedelta
        from django.utils import timezone
        
        today = timezone.now().date()
        
        if self.value() == 'today':
            return queryset.filter(pub_date__date=today)
        elif self.value() == 'week':
            start = today - timedelta(days=today.weekday())
            return queryset.filter(pub_date__date__gte=start)
        elif self.value() == 'month':
            return queryset.filter(
                pub_date__year=today.year,
                pub_date__month=today.month
            )
        elif self.value() == 'quarter':
            quarter_start = date(today.year, (today.month - 1) // 3 * 3 + 1, 1)
            return queryset.filter(pub_date__date__gte=quarter_start)
        elif self.value() == 'year':
            return queryset.filter(pub_date__year=today.year)
        return queryset
```

### Related Field Filter

```python
class AuthorCountryFilter(SimpleListFilter):
    """Filter articles by author's country."""
    title = 'author country'
    parameter_name = 'country'
    
    def lookups(self, request, model_admin):
        countries = Author.objects.values_list(
            'profile__country', flat=True
        ).distinct()
        return [(c, c) for c in countries if c]
    
    def queryset(self, request, queryset):
        if self.value():
            return queryset.filter(author__profile__country=self.value())
        return queryset
```

## Common Pitfalls

1. **Returning None from queryset() when self.value() is None**: The queryset() method must return the full queryset when no filter is selected (self.value() is None). Returning None causes a 500 error.

2. **Using too many list_filter entries**: Each filter generates a sidebar section. More than 5-6 filters overwhelm the interface. Group related filters into a single custom SimpleListFilter instead.

3. **Not using RelatedOnlyFieldListFilter for ForeignKey fields**: The default shows all related objects, including those with zero associations. RelatedOnlyFieldListFilter is almost always what you want.

## Best Practices

1. **Place the most-used filters first**: list_filter renders filters in order. Put the filters admins use most often at the top of the sidebar.

2. **Use meaningful titles and labels**: Set the `title` attribute on SimpleListFilter to a clear, lowercase description so the sidebar reads naturally.

3. **Cache expensive lookups()**: If lookups() runs aggregation queries, cache the results to avoid recalculating on every page load.

## Summary

- list_filter adds sidebar filters to the change list.
- Built-in filters work for standard field types, ForeignKey, and ManyToMany.
- SimpleListFilter requires implementing lookups() and queryset().
- RelatedOnlyFieldListFilter limits options to objects with existing relations.
- Custom filters can use request context and database annotations.

## Code Examples

**Custom SimpleListFilter implementation for filtering by decade**

```python
from django.contrib import admin
from datetime import date

class DecadeBornListFilter(admin.SimpleListFilter):
    title = 'decade born'
    parameter_name = 'decade'

    def lookups(self, request, model_admin):
        return [
            ('80s', 'in the eighties'),
            ('90s', 'in the nineties'),
        ]

    def queryset(self, request, queryset):
        if self.value() == '80s':
            return queryset.filter(
                birthday__gte=date(1980, 1, 1),
                birthday__lte=date(1989, 12, 31),
            )
        if self.value() == '90s':
            return queryset.filter(
                birthday__gte=date(1990, 1, 1),
                birthday__lte=date(1999, 12, 31),
            )
```


## Resources

- [List Filters](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.list_filter) — List filter documentation

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*