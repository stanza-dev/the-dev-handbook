---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-wizard-storage-backends"
---

# Session vs Cookie Wizard

## Introduction

django-formtools provides two wizard views with different storage backends: SessionWizardView stores step data in the server session, while CookieWizardView stores it in signed cookies. Each has trade-offs for scalability, file handling, and security.

## Key Concepts

- **SessionWizardView**: Stores wizard data server-side in the Django session backend.
- **CookieWizardView**: Stores wizard data in signed (not encrypted) browser cookies.
- **file_storage**: Required attribute for wizards that handle file uploads.

## Real World Context

Stateless deployments (like containers behind a load balancer) may not share sessions across instances. CookieWizardView avoids this problem since data travels with the request. However, cookie size limits (4KB per cookie) make it unsuitable for large forms or file uploads.

## Deep Dive

### SessionWizardView

The most common choice. Data is stored server-side in whatever session backend you configured:

```python
from formtools.wizard.views import SessionWizardView


class RegistrationWizard(SessionWizardView):
    form_list = [
        ('info', PersonalInfoForm),
        ('address', AddressForm),
        ('confirm', ConfirmationForm),
    ]
    template_name = 'wizard/step.html'

    def done(self, form_list, **kwargs):
        data = {}
        for form in form_list:
            data.update(form.cleaned_data)
        User.objects.create_user(**data)
        return redirect('welcome')
```

Session storage handles large data and file references without issues. The session backend can be database, cache, or file-based.

### CookieWizardView

Data is stored in signed cookies on the client side:

```python
from formtools.wizard.views import CookieWizardView


class FeedbackWizard(CookieWizardView):
    form_list = [
        ('rating', RatingForm),
        ('comments', CommentsForm),
    ]
    template_name = 'wizard/feedback.html'

    def done(self, form_list, **kwargs):
        data = {}
        for form in form_list:
            data.update(form.cleaned_data)
        Feedback.objects.create(**data)
        return redirect('thanks')
```

Cookies are cryptographically signed using Django's SECRET_KEY, so users cannot tamper with the data. However, the data is not encrypted — users can read (but not modify) it.

### File Uploads in Wizards

File uploads require server-side storage since files cannot fit in cookies. Set file_storage on any wizard that accepts files:

```python
from django.core.files.storage import FileSystemStorage

wizard_storage = FileSystemStorage(
    location='/tmp/wizard_uploads'
)


class DocumentWizard(SessionWizardView):
    form_list = [
        ('info', DocumentInfoForm),
        ('upload', DocumentUploadForm),
        ('review', ReviewForm),
    ]
    file_storage = wizard_storage

    def done(self, form_list, form_dict, **kwargs):
        upload_form = form_dict['upload']
        uploaded_file = upload_form.cleaned_data['document']
        # Move from temp storage to permanent location
        Document.objects.create(
            title=form_dict['info'].cleaned_data['title'],
            file=uploaded_file,
        )
        return redirect('document_list')
```

Temporary files in wizard_storage are cleaned up when the wizard completes. For production, consider using a cloud storage backend.

### Choosing Between Session and Cookie

Here is a comparison to guide your choice:

```python
# Session: Best for most cases
# + Handles large data and files
# + Data stays server-side
# - Requires shared sessions in multi-server setups

# Cookie: Best for stateless deployments
# + No server state needed
# + Works with load balancers out of the box
# - 4KB size limit per cookie
# - Data is readable (signed, not encrypted)
# - Cannot handle file uploads directly
```

## Common Pitfalls

1. **Forgetting file_storage** — If your wizard includes a FileField and you don't set file_storage, uploaded files are lost between steps. Django raises no error; the file simply disappears.
2. **Cookie size overflow** — CookieWizardView silently truncates data that exceeds the cookie size limit. Forms with many text fields can hit this limit unexpectedly.

## Best Practices

1. **Default to SessionWizardView** — Unless you have a specific reason for stateless storage, session-based wizards are simpler and handle all edge cases including files.
2. **Clean up wizard temp files** — Set up a periodic task (cron or Celery) to remove orphaned temp files from your wizard file_storage location, in case users abandon the wizard mid-flow.

## Summary

- SessionWizardView stores data server-side; best for most applications.
- CookieWizardView stores data in signed cookies; best for stateless deployments.
- File uploads always require server-side file_storage, regardless of wizard type.
- Cookies are signed but not encrypted — do not store sensitive data in CookieWizardView.

## Resources

- [Form Wizard Storage Backends](https://django-formtools.readthedocs.io/en/latest/wizard.html#handling-files) — File handling in form wizards

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*