---
source_course: "react-modern-patterns"
source_lesson: "react-server-function-security"
---

# Server Function Security

## Introduction

Every Server Function you write becomes a publicly accessible HTTP endpoint. This is not a limitation — it is a design choice that enables seamless client-server communication. But it means you must treat Server Functions with the same security rigor as API routes. Authentication, authorization, input validation, and rate limiting are not optional. This lesson covers the security model and best practices for keeping your Server Functions safe.

## Key Concepts

- **Public Endpoint**: Any function marked `"use server"` can be called by any client that knows the endpoint, regardless of what component renders the form.
- **Input Validation**: All arguments (FormData, direct parameters) must be validated on the server because they can be tampered with.
- **Authentication**: Always verify the user's identity inside the Server Function, not in the Client Component.
- **Authorization**: Check that the authenticated user has permission to perform the specific action.

## Real World Context

Imagine an admin panel where only administrators can delete users. A naive implementation might hide the "Delete User" button for non-admins on the client. But the Server Function itself is a public endpoint — anyone could call `deleteUser(userId)` directly using browser dev tools or a script. The Server Function must independently verify that the caller is an authenticated administrator, regardless of what the UI shows.

## Deep Dive

Here is a secure Server Function pattern:

```tsx
"use server";

import { auth } from "@/lib/auth";
import { z } from "zod";
import { db } from "@/lib/db";
import { revalidatePath } from "next/cache";

const DeleteUserSchema = z.object({
  userId: z.string().uuid(),
});

export async function deleteUser(formData: FormData) {
  // 1. Authenticate
  const session = await auth.getSession();
  if (!session) {
    throw new Error("Unauthorized");
  }

  // 2. Authorize
  if (session.user.role !== "ADMIN") {
    throw new Error("Forbidden: Admin access required");
  }

  // 3. Validate input
  const parsed = DeleteUserSchema.safeParse({
    userId: formData.get("userId"),
  });

  if (!parsed.success) {
    return { error: "Invalid user ID format" };
  }

  // 4. Prevent self-deletion
  if (parsed.data.userId === session.user.id) {
    return { error: "Cannot delete your own account" };
  }

  // 5. Execute
  await db.user.delete({ where: { id: parsed.data.userId } });
  revalidatePath("/admin/users");
  return { error: null };
}
```

Key security layers in every Server Function:

```tsx
// Pattern: Auth + Authorize + Validate + Execute
export async function secureAction(formData: FormData) {
  // Step 1: Who is calling?
  const session = await getSession();
  if (!session) return { error: "Not logged in" };

  // Step 2: Are they allowed?
  const canDo = await checkPermission(session.userId, "action:perform");
  if (!canDo) return { error: "Not authorized" };

  // Step 3: Is the input valid?
  const input = schema.safeParse(Object.fromEntries(formData));
  if (!input.success) return { error: "Invalid input" };

  // Step 4: Do the thing
  await performAction(input.data);
  return { error: null };
}
```

Encrypted closures provide an additional layer. When a Server Function is defined inline in a Server Component and closes over server data, React encrypts the closed-over values so they cannot be read or tampered with on the client. However, you should not rely on this as your only security measure.

## Common Pitfalls

1. **Trusting client-side validation alone** — Even if you validate inputs in the Client Component before calling the Server Function, the function can be called directly with any arguments. Always re-validate on the server.
2. **Checking permissions in the UI instead of the Server Function** — Hiding a button does not prevent the Server Function from being called. Authorization must happen inside the function itself.

## Best Practices

1. **Follow the Auth-Authorize-Validate-Execute pattern** — Every Server Function should authenticate the user, check authorization, validate all inputs, and only then execute the business logic.
2. **Use Zod schemas for input validation** — Define a schema for every Server Function's expected input and parse with `safeParse` to get structured errors without throwing.

## Summary

- Server Functions are public HTTP endpoints; always authenticate and authorize inside them, never rely on UI-level restrictions.
- Validate all inputs with a schema library like Zod because FormData can be manipulated by any client.
- Follow the Auth-Authorize-Validate-Execute pattern for every Server Function to ensure comprehensive security.

## Code Examples

**Secure Server Function with authentication and Zod validation**

```tsx
"use server";

import { auth } from "@/lib/auth";
import { z } from "zod";
import { db } from "@/lib/db";

const UpdateProfileSchema = z.object({
  name: z.string().min(2).max(50),
  bio: z.string().max(500).optional(),
});

export async function updateProfile(prevState: any, formData: FormData) {
  // 1. Authenticate
  const session = await auth.getSession();
  if (!session) return { error: "Please log in" };

  // 2. Validate
  const parsed = UpdateProfileSchema.safeParse({
    name: formData.get("name"),
    bio: formData.get("bio"),
  });
  if (!parsed.success) return { error: parsed.error.issues[0].message };

  // 3. Execute (user can only update own profile)
  await db.user.update({
    where: { id: session.user.id },
    data: parsed.data,
  });
  return { error: null, success: true };
}
```


## Resources

- [Server Functions Security](https://react.dev/reference/rsc/server-functions#security) — Official security guidance for Server Functions

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*