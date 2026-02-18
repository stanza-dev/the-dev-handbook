---
source_course: "php-security"
source_lesson: "php-security-file-upload-security"
---

# Secure File Upload Handling

File uploads are a common attack vector. Improper handling can lead to remote code execution, path traversal, and denial of service.

## Dangers of File Uploads

1. **Remote Code Execution**: Uploading PHP files that get executed
2. **Path Traversal**: Using `../` to write outside upload directory
3. **MIME Spoofing**: Disguising malicious files with fake extensions
4. **Denial of Service**: Uploading huge files to exhaust storage

## Secure Upload Handler

```php
<?php
class SecureFileUpload
{
    private const ALLOWED_TYPES = [
        'image/jpeg' => 'jpg',
        'image/png' => 'png',
        'image/gif' => 'gif',
        'application/pdf' => 'pdf',
    ];
    
    private const MAX_SIZE = 5 * 1024 * 1024;  // 5MB
    
    public function __construct(
        private string $uploadDir
    ) {
        // Ensure upload directory is outside webroot
        if (str_starts_with(realpath($this->uploadDir), $_SERVER['DOCUMENT_ROOT'])) {
            throw new RuntimeException('Upload directory must be outside webroot');
        }
    }
    
    public function upload(array $file): string
    {
        // 1. Check for upload errors
        if ($file['error'] !== UPLOAD_ERR_OK) {
            throw new UploadException($this->getUploadErrorMessage($file['error']));
        }
        
        // 2. Validate file size
        if ($file['size'] > self::MAX_SIZE) {
            throw new UploadException('File too large');
        }
        
        // 3. Validate MIME type (check actual content, not extension)
        $finfo = new finfo(FILEINFO_MIME_TYPE);
        $mimeType = $finfo->file($file['tmp_name']);
        
        if (!isset(self::ALLOWED_TYPES[$mimeType])) {
            throw new UploadException('Invalid file type');
        }
        
        // 4. Generate safe filename (never use original filename!)
        $extension = self::ALLOWED_TYPES[$mimeType];
        $newFilename = bin2hex(random_bytes(16)) . '.' . $extension;
        
        // 5. Move to secure location
        $destination = $this->uploadDir . '/' . $newFilename;
        
        if (!move_uploaded_file($file['tmp_name'], $destination)) {
            throw new UploadException('Failed to save file');
        }
        
        // 6. Set restrictive permissions
        chmod($destination, 0644);
        
        return $newFilename;
    }
    
    private function getUploadErrorMessage(int $error): string
    {
        return match($error) {
            UPLOAD_ERR_INI_SIZE => 'File exceeds upload_max_filesize',
            UPLOAD_ERR_FORM_SIZE => 'File exceeds form MAX_FILE_SIZE',
            UPLOAD_ERR_PARTIAL => 'File only partially uploaded',
            UPLOAD_ERR_NO_FILE => 'No file uploaded',
            UPLOAD_ERR_NO_TMP_DIR => 'Missing temp folder',
            UPLOAD_ERR_CANT_WRITE => 'Failed to write file',
            default => 'Unknown upload error',
        };
    }
}
```

## Serving Uploaded Files Securely

```php
<?php
class SecureFileServer
{
    public function serve(string $filename): void
    {
        // Validate filename format
        if (!preg_match('/^[a-f0-9]{32}\.(jpg|png|gif|pdf)$/', $filename)) {
            http_response_code(400);
            exit('Invalid filename');
        }
        
        $path = '/var/uploads/' . $filename;  // Outside webroot
        
        if (!file_exists($path)) {
            http_response_code(404);
            exit('File not found');
        }
        
        // Determine content type
        $finfo = new finfo(FILEINFO_MIME_TYPE);
        $mimeType = $finfo->file($path);
        
        // Security headers
        header('Content-Type: ' . $mimeType);
        header('Content-Disposition: inline; filename="' . $filename . '"');
        header('X-Content-Type-Options: nosniff');
        header('Content-Security-Policy: default-src \'none\'');
        
        // Serve file
        readfile($path);
    }
}
```

## Image-Specific Validation

```php
<?php
function validateImage(string $filepath): bool
{
    // Actually try to load as image
    $imageInfo = @getimagesize($filepath);
    
    if ($imageInfo === false) {
        return false;  // Not a valid image
    }
    
    // Check image type
    $allowedTypes = [IMAGETYPE_JPEG, IMAGETYPE_PNG, IMAGETYPE_GIF];
    if (!in_array($imageInfo[2], $allowedTypes, true)) {
        return false;
    }
    
    // Re-encode to strip any embedded code
    $image = match($imageInfo[2]) {
        IMAGETYPE_JPEG => imagecreatefromjpeg($filepath),
        IMAGETYPE_PNG => imagecreatefrompng($filepath),
        IMAGETYPE_GIF => imagecreatefromgif($filepath),
    };
    
    // Save re-encoded image (removes any embedded PHP)
    imagejpeg($image, $filepath, 90);
    imagedestroy($image);
    
    return true;
}
```

## Resources

- [File Uploads](https://www.php.net/manual/en/features.file-upload.php) — PHP file upload handling

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*