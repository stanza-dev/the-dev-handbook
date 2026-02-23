---
source_course: "django-foundations"
source_lesson: "django-foundations-crud-operations"
---

# CRUD Operations with the ORM

CRUD stands for **Create, Read, Update, Delete**—the four basic database operations. Django's ORM makes these operations feel natural and Pythonic.

## Using the Django Shell

The Django shell loads your project settings:

```bash
python manage.py shell
```

```python
>>> from polls.models import Question, Choice
>>> from django.utils import timezone
```

## Create

There are several ways to create records:

### Using `save()`

The most explicit way to create a record is to instantiate the model and then call `save()` to write it to the database.

```python
# Create an instance
q = Question(question_text="What's new?", pub_date=timezone.now())

# Save to database
q.save()

# Now it has an ID
print(q.id)  # 1
```

### Using `create()`

A more concise alternative is `objects.create()`, which instantiates and saves the object in a single step.

```python
# Create and save in one step
q = Question.objects.create(
    question_text="What's your favorite color?",
    pub_date=timezone.now()
)
```

### Creating Related Objects

When creating objects that reference another model via a ForeignKey, you can use the related manager or set the foreign key directly.

```python
# Create choices for a question
q = Question.objects.get(pk=1)

# Method 1: Using create on the related manager
q.choice_set.create(choice_text='Red', votes=0)
q.choice_set.create(choice_text='Blue', votes=0)

# Method 2: Setting the foreign key directly
c = Choice(question=q, choice_text='Green', votes=0)
c.save()
```

## Read

Django provides powerful query methods:

### Retrieving All Objects

The `all()` method returns a QuerySet containing every record in the table.

```python
# Get all questions
questions = Question.objects.all()
# <QuerySet [<Question: What's new?>, <Question: What's your favorite color?>]>
```

### Retrieving Specific Objects

Use `get()` when you need exactly one object. It raises an exception if zero or multiple records match.

```python
# Get by primary key
q = Question.objects.get(pk=1)

# Get by field value
q = Question.objects.get(question_text="What's new?")

# get() raises DoesNotExist if no match
# get() raises MultipleObjectsReturned if multiple matches
```

### Filtering

The `filter()` method returns a QuerySet of objects matching the given conditions. You can chain multiple conditions for AND logic.

```python
# Filter returns a QuerySet
Question.objects.filter(pub_date__year=2024)

# Multiple conditions (AND)
Question.objects.filter(
    pub_date__year=2024,
    question_text__startswith='What'
)

# Exclude
Question.objects.exclude(pub_date__year=2023)
```

### Field Lookups

Use double underscores for lookups:

```python
# Exact match (default)
Question.objects.filter(id=1)
Question.objects.filter(id__exact=1)  # Same as above

# Case-insensitive exact
Question.objects.filter(question_text__iexact="what's new?")

# Contains
Question.objects.filter(question_text__contains='favorite')
Question.objects.filter(question_text__icontains='FAVORITE')  # Case-insensitive

# Starts/ends with
Question.objects.filter(question_text__startswith='What')
Question.objects.filter(question_text__endswith='?')

# Date lookups
Question.objects.filter(pub_date__year=2024)
Question.objects.filter(pub_date__month=6)
Question.objects.filter(pub_date__gte=timezone.now())  # Greater than or equal

# In a list
Question.objects.filter(id__in=[1, 3, 5])
```

### Ordering

Use `order_by()` to sort QuerySet results. Prefix a field name with `-` for descending order.

```python
# Ascending
Question.objects.order_by('pub_date')

# Descending
Question.objects.order_by('-pub_date')

# Multiple fields
Question.objects.order_by('pub_date', 'question_text')
```

## Update

### Updating a Single Object

To update a single object, modify its attributes and call `save()` to persist the changes.

```python
q = Question.objects.get(pk=1)
q.question_text = "What's up?"
q.save()
```

### Updating Multiple Objects

For bulk updates, use `QuerySet.update()` which runs a single SQL query instead of saving each object individually.

```python
# Update all choices to 0 votes
Choice.objects.filter(question__pk=1).update(votes=0)
```

## Delete

### Deleting a Single Object

Call `delete()` on a model instance to remove it from the database. Django returns a tuple showing what was deleted, including any cascaded deletions.

```python
q = Question.objects.get(pk=1)
q.delete()
# (3, {'polls.Choice': 2, 'polls.Question': 1})
# Returns count and objects deleted (including cascaded)
```

### Deleting Multiple Objects

You can also call `delete()` on a QuerySet to remove multiple objects at once.

```python
# Delete all choices with 0 votes
Choice.objects.filter(votes=0).delete()
```

## QuerySets are Lazy

QuerySets don't hit the database until evaluated:

```python
# No database query yet
qs = Question.objects.filter(pub_date__year=2024)

# Database is queried when:
list(qs)           # Converting to list
for q in qs:       # Iterating
    print(q)
qs[0]              # Indexing
print(qs)          # Printing
```

## Common Pitfalls

- **Using `get()` when multiple objects might match**: `get()` raises `MultipleObjectsReturned` if more than one object matches. Use `filter()` when you expect multiple results.
- **Forgetting that QuerySets are lazy**: No database query runs until the QuerySet is evaluated (iterated, sliced, or converted to a list).
- **Not handling `DoesNotExist`**: Always wrap `get()` calls in try/except or use `get_object_or_404()` in views.

## Best Practices

- **Use `create()` for single-step creation** instead of instantiating and calling `save()` separately.
- **Use `update()` for bulk updates** instead of looping through objects and calling `save()` on each one.
- **Chain QuerySet methods** for readable queries: `Question.objects.filter(...).exclude(...).order_by(...)`.

## Summary

- **Create** objects with `Model.objects.create()` or instantiate and call `.save()`
- **Read** objects with `all()`, `get()` (single), or `filter()` (multiple)
- **Update** by modifying attributes and calling `.save()`, or use `QuerySet.update()` for bulk updates
- **Delete** with `.delete()` on an instance or QuerySet
- QuerySets are **lazy** and support chaining, field lookups (double underscores), and ordering

## Code Examples

**Complete CRUD operations using Django ORM: create, read, filter, update, and delete**

```python
from polls.models import Question
from django.utils import timezone

# Create
q = Question.objects.create(
    question_text="What's your favorite color?",
    pub_date=timezone.now()
)

# Read
all_questions = Question.objects.all()
recent = Question.objects.filter(pub_date__year=2024)

# Update
q.question_text = "What's your favorite language?"
q.save()

# Delete
q.delete()
```


## Resources

- [Making Queries](https://docs.djangoproject.com/en/6.0/topics/db/queries/) — Official guide to querying the database with Django's ORM

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*