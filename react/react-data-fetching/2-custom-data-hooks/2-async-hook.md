---
source_course: "react-data-fetching"
source_lesson: "react-data-fetching-use-async-hook"
---

# Generic useAsync Hook

A more flexible hook that works with any async function, not just fetch.

## The Hook

```jsx
import { useState, useCallback } from 'react';

function useAsync(asyncFunction, immediate = true) {
  const [status, setStatus] = useState('idle');
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);

  const execute = useCallback(
    async (...args) => {
      setStatus('pending');
      setData(null);
      setError(null);

      try {
        const response = await asyncFunction(...args);
        setData(response);
        setStatus('success');
        return response;
      } catch (err) {
        setError(err);
        setStatus('error');
        throw err;
      }
    },
    [asyncFunction]
  );

  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);

  return {
    execute,
    status,
    data,
    error,
    isIdle: status === 'idle',
    isPending: status === 'pending',
    isSuccess: status === 'success',
    isError: status === 'error',
  };
}
```

## Usage Examples

### Fetch User

```jsx
const fetchUser = async (id) => {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
};

function UserProfile({ userId }) {
  const { data: user, isPending, isError, error } = useAsync(
    () => fetchUser(userId),
    true
  );

  if (isPending) return <Spinner />;
  if (isError) return <Error error={error} />;
  return <Profile user={user} />;
}
```

### Form Submission

```jsx
const submitForm = async (data) => {
  const response = await fetch('/api/submit', {
    method: 'POST',
    body: JSON.stringify(data),
  });
  return response.json();
};

function ContactForm() {
  const { execute, isPending, isSuccess, isError, error } = useAsync(
    submitForm,
    false // Don't run immediately
  );

  const handleSubmit = async (formData) => {
    await execute(formData);
  };

  if (isSuccess) return <p>Form submitted successfully!</p>;

  return (
    <form onSubmit={handleSubmit}>
      {/* fields */}
      <button disabled={isPending}>
        {isPending ? 'Submitting...' : 'Submit'}
      </button>
      {isError && <p className="error">{error.message}</p>}
    </form>
  );
}
```

### File Upload

```jsx
const uploadFile = async (file) => {
  const formData = new FormData();
  formData.append('file', file);
  
  const response = await fetch('/api/upload', {
    method: 'POST',
    body: formData,
  });
  return response.json();
};

function FileUpload() {
  const { execute, isPending, data, error } = useAsync(uploadFile, false);

  const handleFileChange = (e) => {
    const file = e.target.files[0];
    if (file) execute(file);
  };

  return (
    <div>
      <input type="file" onChange={handleFileChange} disabled={isPending} />
      {isPending && <p>Uploading...</p>}
      {data && <p>Uploaded: {data.url}</p>}
      {error && <p>Error: {error.message}</p>}
    </div>
  );
}
```

---

> 📘 *This lesson is part of the [React Data Fetching Patterns](https://stanza.dev/courses/react-data-fetching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*