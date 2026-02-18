---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-custom-admin-forms"
---

# Custom Admin Forms

Customize the forms used in the admin change view for validation and advanced functionality.

## Basic Form Customization

```python
from django import forms
from django.contrib import admin
from .models import Article


class ArticleAdminForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = '__all__'
        widgets = {
            'body': forms.Textarea(attrs={
                'rows': 20,
                'cols': 80,
                'class': 'vLargeTextField',
            }),
            'summary': forms.Textarea(attrs={'rows': 3}),
        }
    
    def clean_title(self):
        title = self.cleaned_data['title']
        if len(title) < 10:
            raise forms.ValidationError('Title must be at least 10 characters.')
        return title
    
    def clean(self):
        cleaned_data = super().clean()
        status = cleaned_data.get('status')
        pub_date = cleaned_data.get('pub_date')
        
        if status == 'published' and not pub_date:
            raise forms.ValidationError(
                'Published articles must have a publication date.'
            )
        
        return cleaned_data


@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    form = ArticleAdminForm
```

## Dynamic Form Based on Request

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    def get_form(self, request, obj=None, **kwargs):
        form = super().get_form(request, obj, **kwargs)
        
        # Add help text dynamically
        form.base_fields['title'].help_text = 'Enter a compelling title'
        
        # Modify queryset for foreign keys
        if not request.user.is_superuser:
            form.base_fields['author'].queryset = User.objects.filter(
                groups__name='Authors'
            )
        
        return form
    
    def get_readonly_fields(self, request, obj=None):
        if obj and obj.status == 'published':
            # Can't edit published articles
            return ['title', 'slug', 'body', 'author']
        return []
```

## Custom Widgets

```python
from django.contrib.admin.widgets import AdminDateWidget, FilteredSelectMultiple
from django.forms.widgets import Select


class ArticleAdminForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = '__all__'
        widgets = {
            'pub_date': AdminDateWidget(),
            'tags': FilteredSelectMultiple(
                verbose_name='Tags',
                is_stacked=False,
            ),
            'category': Select(attrs={'class': 'select2'}),
        }
```

## Rich Text Editor Integration

```python
# Using django-ckeditor or similar
from ckeditor.widgets import CKEditorWidget

class ArticleAdminForm(forms.ModelForm):
    body = forms.CharField(widget=CKEditorWidget())
    
    class Meta:
        model = Article
        fields = '__all__'
```

## Conditional Fields

```python
class ArticleAdminForm(forms.ModelForm):
    schedule_publish = forms.BooleanField(
        required=False,
        help_text='Schedule this article for future publication'
    )
    
    class Meta:
        model = Article
        fields = '__all__'
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        
        # Pre-check if article has future pub_date
        if self.instance.pk:
            from django.utils import timezone
            if self.instance.pub_date and self.instance.pub_date > timezone.now():
                self.fields['schedule_publish'].initial = True
    
    class Media:
        js = ('admin/js/article_form.js',)  # JS to show/hide fields
```

## Form with Extra Fields

```python
class ArticleAdminForm(forms.ModelForm):
    # Extra field not in model
    send_notification = forms.BooleanField(
        required=False,
        label='Send notification',
        help_text='Notify subscribers about this article'
    )
    
    class Meta:
        model = Article
        fields = '__all__'


@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    form = ArticleAdminForm
    
    def save_model(self, request, obj, form, change):
        super().save_model(request, obj, form, change)
        
        if form.cleaned_data.get('send_notification'):
            send_article_notification(obj)
```

## Resources

- [ModelAdmin.form](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.form) — Custom admin forms reference

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*