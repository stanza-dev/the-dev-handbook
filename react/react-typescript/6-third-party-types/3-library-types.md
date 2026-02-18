---
source_course: "react-typescript"
source_lesson: "react-typescript-form-library-types"
---

# Form Library Types

Type-safe forms with React Hook Form and Zod.

## React Hook Form Basics

```tsx
import { useForm, SubmitHandler } from 'react-hook-form';

type FormData = {
  email: string;
  password: string;
  rememberMe: boolean;
};

function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors }
  } = useForm<FormData>();

  const onSubmit: SubmitHandler<FormData> = (data) => {
    // data is fully typed
    console.log(data.email, data.password);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email', { required: 'Email required' })} />
      {errors.email && <span>{errors.email.message}</span>}
      
      <input
        type="password"
        {...register('password', { minLength: 8 })}
      />
      
      <input type="checkbox" {...register('rememberMe')} />
      
      <button type="submit">Login</button>
    </form>
  );
}
```

## With Zod Resolver

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Min 8 characters'),
  age: z.number().min(18, 'Must be 18+'),
});

// Infer type from schema
type FormData = z.infer<typeof schema>;

function SignupForm() {
  const {
    register,
    handleSubmit,
    formState: { errors }
  } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  const onSubmit = (data: FormData) => {
    // data is validated and typed
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} />
      {errors.email && <span>{errors.email.message}</span>}
      
      <input type="password" {...register('password')} />
      {errors.password && <span>{errors.password.message}</span>}
      
      <input
        type="number"
        {...register('age', { valueAsNumber: true })}
      />
      {errors.age && <span>{errors.age.message}</span>}
      
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

## Typed Form Components

```tsx
import { UseFormRegister, FieldErrors, Path } from 'react-hook-form';

type InputProps<T extends Record<string, any>> = {
  name: Path<T>;
  register: UseFormRegister<T>;
  errors: FieldErrors<T>;
  label: string;
};

function Input<T extends Record<string, any>>({
  name,
  register,
  errors,
  label
}: InputProps<T>) {
  const error = errors[name];
  
  return (
    <div>
      <label>{label}</label>
      <input {...register(name)} />
      {error && <span>{error.message as string}</span>}
    </div>
  );
}
```

---

> 📘 *This lesson is part of the [React TypeScript Mastery](https://stanza.dev/courses/react-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*