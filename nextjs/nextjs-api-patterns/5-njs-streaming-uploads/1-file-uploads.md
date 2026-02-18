---
source_course: "nextjs-api-patterns"
source_lesson: "nextjs-api-patterns-njs-file-uploads"
---

# File Uploads in Next.js

Route Handlers can handle `multipart/form-data` for file uploads.

## Using FormData

```typescript
// app/api/upload/route.ts
import { writeFile } from 'fs/promises';
import { join } from 'path';

export async function POST(request: Request) {
  const formData = await request.formData();
  const file = formData.get('file') as File;

  if (!file) {
    return Response.json({ error: 'No file uploaded' }, { status: 400 });
  }

  const bytes = await file.arrayBuffer();
  const buffer = Buffer.from(bytes);

  // Save to public/uploads (for demo only, use cloud storage in production)
  const path = join(process.cwd(), 'public', 'uploads', file.name);
  await writeFile(path, buffer);

  return Response.json({ success: true, filename: file.name });
}
```

## Client-Side Upload

```typescript
'use client';

async function uploadFile(file: File) {
  const formData = new FormData();
  formData.append('file', file);

  const response = await fetch('/api/upload', {
    method: 'POST',
    body: formData,
  });

  return response.json();
}
```

## Production Considerations

- Use cloud storage (S3, Cloudflare R2, Vercel Blob).
- Validate file types and sizes.
- Scan for malware.

---

> 📘 *This lesson is part of the [Next.js API Routes & Patterns](https://stanza.dev/courses/nextjs-api-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*