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

## Real World Context

File uploads are everywhere: user avatars, document attachments, CSV imports, image galleries. Without proper handling, uploads can exhaust server memory, allow malicious files, or corrupt data. NestJS's multer integration handles streaming, size limits, and file type filtering.

## Deep Dive

### Single File Upload

Wrap a controller method with `FileInterceptor` to handle a single file field from a multipart form.

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

The string `'file'` passed to `FileInterceptor` must match the form field name used by the client.

### Multiple Files

Use `FilesInterceptor` with a max count argument to accept multiple files under the same field name.

```typescript
@Post('uploads')
@UseInterceptors(FilesInterceptor('files', 10))
uploadFiles(@UploadedFiles() files: Express.Multer.File[]) {
  return files.map(f => f.originalname);
}
```

The second argument (`10`) limits the number of files accepted per request.

### File Validation

Pass a `ParseFilePipe` with validators to `@UploadedFile()` to enforce size and type constraints.

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

Validation runs before the handler executes, so rejected files never reach your business logic.

### Custom Storage

Configure `diskStorage` to control where files are saved and how filenames are generated.

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

Generating unique filenames with timestamps prevents overwrites when users upload files with identical names.

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

- NestJS handles file uploads via FileInterceptor built on top of multer
- Use ParseFilePipe with MaxFileSizeValidator and FileTypeValidator for security
- Configure custom storage destinations with diskStorage or cloud providers
- Always validate file size and type to prevent malicious uploads
- Use FilesInterceptor for handling multiple file uploads

## Code Examples

**Handling file uploads with @UseInterceptors and FileInterceptor from @nestjs/platform-express**

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


## Resources

- [File Upload](https://docs.nestjs.com/techniques/file-upload) — Official file upload guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*