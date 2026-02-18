---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-model-inheritance"
---

# Model Inheritance Patterns

Django supports three types of model inheritance, each with different use cases and database implications.

## 1. Abstract Base Classes

Share common fields without creating a database table:

```python
class BaseContent(models.Model):
    """Abstract base - no table created."""
    title = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    is_published = models.BooleanField(default=False)

    class Meta:
        abstract = True
        ordering = ['-created_at']

    def publish(self):
        self.is_published = True
        self.save()


class Article(BaseContent):
    """Creates table: appname_article with all fields."""
    body = models.TextField()
    author = models.ForeignKey('Author', on_delete=models.CASCADE)


class Video(BaseContent):
    """Creates table: appname_video with all fields."""
    url = models.URLField()
    duration = models.IntegerField(help_text='Duration in seconds')
```

**Database Result:**
- No `basecontent` table
- `article` table with: title, slug, created_at, updated_at, is_published, body, author_id
- `video` table with: title, slug, created_at, updated_at, is_published, url, duration

## 2. Multi-table Inheritance

Each model gets its own table, linked by foreign key:

```python
class Place(models.Model):
    """Creates table: appname_place."""
    name = models.CharField(max_length=100)
    address = models.CharField(max_length=200)


class Restaurant(Place):
    """Creates table: appname_restaurant with FK to place."""
    serves_pizza = models.BooleanField(default=False)
    serves_pasta = models.BooleanField(default=False)


class Cafe(Place):
    """Creates table: appname_cafe with FK to place."""
    serves_coffee = models.BooleanField(default=True)
    has_wifi = models.BooleanField(default=True)
```

**Database Result:**
- `place` table: id, name, address
- `restaurant` table: place_ptr_id (FK), serves_pizza, serves_pasta
- `cafe` table: place_ptr_id (FK), serves_coffee, has_wifi

**Usage:**
```python
# Create a restaurant
r = Restaurant.objects.create(
    name='Pizza Palace',
    address='123 Main St',
    serves_pizza=True
)

# Access parent fields directly
print(r.name)  # 'Pizza Palace'

# Access from parent
place = Place.objects.get(pk=r.pk)
print(place.restaurant.serves_pizza)  # True
```

## 3. Proxy Models

Same table, different Python behavior:

```python
class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    is_staff = models.BooleanField(default=False)

    def get_full_name(self):
        return f'{self.first_name} {self.last_name}'


class StaffMember(Person):
    """No new table - uses Person table."""
    class Meta:
        proxy = True
        ordering = ['last_name', 'first_name']

    def get_full_name(self):
        return f'{self.last_name}, {self.first_name}'  # Different format

    objects = StaffManager()  # Custom manager
```

**Use Cases for Proxy Models:**
- Different default ordering
- Custom managers with filtered querysets
- Different methods or properties
- Different admin configurations

## Choosing the Right Pattern

| Pattern | Use When | Trade-offs |
|---------|----------|------------|
| **Abstract** | Sharing fields/methods, no common queries | No polymorphic queries |
| **Multi-table** | Need to query parent type, true polymorphism | Extra JOINs, more complex |
| **Proxy** | Same data, different behavior | Can't add fields |

## Mixing Patterns

```python
class Timestamp(models.Model):
    """Abstract mixin."""
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        abstract = True


class Content(Timestamp):
    """Concrete model with timestamps."""
    title = models.CharField(max_length=200)


class PublishedContent(Content):
    """Proxy with custom manager."""
    class Meta:
        proxy = True

    objects = PublishedManager()  # Only returns published items
```

## Resources

- [Model Inheritance](https://docs.djangoproject.com/en/6.0/topics/db/models/#model-inheritance) — Official guide to model inheritance patterns

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*