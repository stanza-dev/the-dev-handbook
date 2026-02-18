---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-file-upload"
---

# File Upload

## Introduction

Many applications need to handle file uploads—images, documents, media. NestJS provides built-in support for multipart/form-data file uploads with Express's multer under the hood.

## Key Concepts

- **@UseInterceptors(FileInterceptor)**: Intercepts single file uploads
- **@UploadedFile()**: Extracts uploaded file from request
- **FilesInterceptor**: Handles multiple files
- **ParseFilePipe**: Validates file type and size

## Deep Dive

### Single File Upload

```typescript
import { FileInterceptor } from '@nestjs/platform-express';
import { UploadedFile, UseInterceptors } from '@nestjs/common';

@Post('upload')
@UseInterceptors(FileInterceptor('file'))
uploadFile(@UploadedFile() file: Express.Multer.File) {
  console.log(file);
  return { filename: file.originalname, size: file.size };
}
```

### Multiple Files

```typescript
@Post('uploads')
@UseInterceptors(FilesInterceptor('files', 10))
uploadFiles(@UploadedFiles() files: Express.Multer.File[]) {
  return files.map(f => f.originalname);
}
```

### File Validation

```typescript
@Post('upload')
@UseInterceptors(FileInterceptor('file'))
uploadFile(
  @UploadedFile(
    new ParseFilePipe({
      validators: [
        new MaxFileSizeValidator({ maxSize: 5 * 1024 * 1024 }), // 5MB
        new FileTypeValidator({ fileType: 'image/jpeg' }),
      ],
    }),
  )
  file: Express.Multer.File,
) {
  return { filename: file.originalname };
}
```

### Custom Storage

```typescript
import { diskStorage } from 'multer';
import { extname } from 'path';

@Post('upload')
@UseInterceptors(FileInterceptor('file', {
  storage: diskStorage({
    destination: './uploads',
    filename: (req, file, cb) => {
      const uniqueName = Date.now() + '-' + Math.round(Math.random() * 1E9);
      cb(null, uniqueName + extname(file.originalname));
    },
  }),
}))
uploadFile(@UploadedFile() file: Express.Multer.File) {
  return { path: file.path };
}
```

## Common Pitfalls

1. **No file size limits**: Default has no limit—easy DoS attack vector.
2. **Not validating file type**: Users can upload malicious files without type checks.
3. **Storing files locally**: Use cloud storage (S3, GCS) in production.

## Best Practices

- Always validate file size and type
- Generate unique filenames to prevent overwrites
- Use cloud storage for production
- Stream large files instead of loading into memory

## Summary

NestJS handles file uploads via FileInterceptor and multer. Use ParseFilePipe for validation, configure storage for custom destinations, and always validate file size and type for security.

## Resources

- [File Upload](https://docs.nestjs.com/techniques/file-upload) — Official file upload guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*