---
source_course: "react-forms"
source_lesson: "react-forms-file-inputs"
---

# File Inputs

File inputs (`<input type="file">`) are **always uncontrolled** in React. You cannot set their value programmatically for security reasons.

## Basic File Input

```jsx
import { useRef } from 'react';

function FileUpload() {
  const fileInputRef = useRef(null);

  function handleSubmit(e) {
    e.preventDefault();
    const file = fileInputRef.current.files[0];
    
    if (file) {
      console.log('File name:', file.name);
      console.log('File size:', file.size, 'bytes');
      console.log('File type:', file.type);
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input type="file" ref={fileInputRef} />
      <button type="submit">Upload</button>
    </form>
  );
}
```

## With onChange Handler

You can react to file selection immediately:

```jsx
function ImagePreview() {
  const [preview, setPreview] = useState(null);

  function handleFileChange(e) {
    const file = e.target.files[0];
    if (file && file.type.startsWith('image/')) {
      const reader = new FileReader();
      reader.onloadend = () => {
        setPreview(reader.result);
      };
      reader.readAsDataURL(file);
    }
  }

  return (
    <div>
      <input
        type="file"
        accept="image/*"
        onChange={handleFileChange}
      />
      {preview && <img src={preview} alt="Preview" />}
    </div>
  );
}
```

## Multiple Files

```jsx
function MultipleFiles() {
  function handleFiles(e) {
    const files = Array.from(e.target.files);
    files.forEach(file => {
      console.log(file.name);
    });
  }

  return (
    <input type="file" multiple onChange={handleFiles} />
  );
}
```

## Uploading with FormData

```jsx
async function uploadFile(file) {
  const formData = new FormData();
  formData.append('file', file);
  formData.append('description', 'My upload');

  const response = await fetch('/api/upload', {
    method: 'POST',
    body: formData  // Don't set Content-Type header!
  });
  
  return response.json();
}
```

⚠️ **Important**: Don't set the `Content-Type` header when uploading files with FormData - the browser sets it automatically with the correct boundary.

---

> 📘 *This lesson is part of the [React Form Mastery](https://stanza.dev/courses/react-forms) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*