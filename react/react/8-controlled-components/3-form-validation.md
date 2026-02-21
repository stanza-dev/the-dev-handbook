---
source_course: "react"
source_lesson: "react-form-validation"
---

# Form Validation Patterns

## Introduction

No form is complete without validation. Users make mistakes, paste incorrect data, and sometimes test boundaries intentionally. React gives you the flexibility to validate on every keystroke, on blur, on submit, or any combination. This lesson covers the most common validation patterns in React, from simple inline checks to schema-based validation with libraries like Zod, and how React 19's form actions change the validation landscape.

## Key Concepts

- **Inline validation**: Checking field values directly in the component using conditional logic.
- **On-blur validation**: Validating a field when the user moves focus away from it, avoiding annoying real-time error messages while typing.
- **Schema validation**: Using a library like Zod to define validation rules declaratively and validate the entire form at once.
- **Server-side validation**: Validating inside Server Functions where you have access to the database (e.g., checking if an email is already taken).

## Real World Context

A checkout form needs different validation strategies for different fields: the email field should validate format on blur, the credit card number should format and validate in real-time (controlled), the expiry date should prevent invalid input entirely, and the full form should validate on submit before charging the card. Combining these strategies gives users helpful feedback without being intrusive.

## Deep Dive

**Inline validation on submit:**

```tsx
function LoginForm() {
  const [form, setForm] = useState({ email: "", password: "" });
  const [errors, setErrors] = useState<Record<string, string>>({});

  const validate = () => {
    const newErrors: Record<string, string> = {};
    if (!form.email.includes("@")) newErrors.email = "Invalid email address";
    if (form.password.length < 8) newErrors.password = "Must be 8+ characters";
    return newErrors;
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    const newErrors = validate();
    setErrors(newErrors);
    if (Object.keys(newErrors).length === 0) {
      submitForm(form);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" value={form.email}
        onChange={(e) => setForm((p) => ({ ...p, email: e.target.value }))} />
      {errors.email && <span className="error">{errors.email}</span>}

      <input name="password" type="password" value={form.password}
        onChange={(e) => setForm((p) => ({ ...p, password: e.target.value }))} />
      {errors.password && <span className="error">{errors.password}</span>}

      <button type="submit">Log In</button>
    </form>
  );
}
```

**On-blur validation:**

```tsx
function BlurValidatedInput({ name, validate, ...props }: {
  name: string;
  validate: (value: string) => string | null;
} & React.InputHTMLAttributes<HTMLInputElement>) {
  const [error, setError] = useState<string | null>(null);

  return (
    <div>
      <input
        name={name}
        onBlur={(e) => setError(validate(e.target.value))}
        onFocus={() => setError(null)}
        {...props}
      />
      {error && <span className="error">{error}</span>}
    </div>
  );
}
```

**Server-side validation with React 19:**

```tsx
"use server";
import { z } from "zod";

const SignupSchema = z.object({
  email: z.string().email("Invalid email"),
  password: z.string().min(8, "Must be at least 8 characters"),
  name: z.string().min(2, "Name too short"),
});

export async function signup(prevState: any, formData: FormData) {
  const parsed = SignupSchema.safeParse(Object.fromEntries(formData));

  if (!parsed.success) {
    return {
      errors: parsed.error.flatten().fieldErrors,
      success: false,
    };
  }

  // Check if email is taken (can only do this server-side)
  const exists = await db.user.findUnique({ where: { email: parsed.data.email } });
  if (exists) {
    return { errors: { email: ["Email already registered"] }, success: false };
  }

  await db.user.create({ data: parsed.data });
  return { errors: {}, success: true };
}
```

## Common Pitfalls

1. **Validating on every keystroke** — Showing error messages while the user is still typing is frustrating. Use on-blur validation for field-level errors and on-submit for the full form.
2. **Only validating on the client** — Client-side validation can be bypassed. Always validate on the server too, especially for security-sensitive operations like authentication and payments.

## Best Practices

1. **Use Zod schemas for consistent validation** — Define your validation rules once as a Zod schema and reuse it on both client and server. The schema serves as documentation for your form's requirements.
2. **Show errors at the right time** — Validate on blur for individual fields (user has finished with the field) and validate the full form on submit. Clear errors when the user starts editing a field again.

## Summary

- Validate on blur for individual fields to avoid annoying real-time errors, and validate the entire form on submit.
- Zod schemas provide declarative, reusable validation rules that work on both client and server.
- React 19 Server Functions enable server-side validation with database checks (like uniqueness) that cannot be done on the client.

## Code Examples

**Form with on-blur validation and submit validation**

```tsx
import { useState } from "react";

function ValidatedForm() {
  const [errors, setErrors] = useState<Record<string, string>>({});

  const validateField = (name: string, value: string) => {
    if (name === "email" && !value.includes("@")) return "Invalid email";
    if (name === "password" && value.length < 8) return "Min 8 characters";
    return "";
  };

  return (
    <form action={async (formData) => {
      const entries = Object.fromEntries(formData);
      const newErrors: Record<string, string> = {};
      for (const [k, v] of Object.entries(entries)) {
        const err = validateField(k, v as string);
        if (err) newErrors[k] = err;
      }
      if (Object.keys(newErrors).length > 0) {
        setErrors(newErrors);
        return;
      }
      await submitForm(entries);
    }}>
      <input name="email" onBlur={(e) =>
        setErrors((p) => ({ ...p, email: validateField("email", e.target.value) }))} />
      {errors.email && <span>{errors.email}</span>}
      <input name="password" type="password" />
      {errors.password && <span>{errors.password}</span>}
      <button type="submit">Submit</button>
    </form>
  );
}
```


## Resources

- [Forms Guide](https://react.dev/learn/reacting-to-input-with-state) — Managing form state and validation in React

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*