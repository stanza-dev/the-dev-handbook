---
source_course: "javascript"
source_lesson: "javascript-try-catch-finally"
---

# Try, Catch, Finally

## Introduction

Errors are inevitable—network failures, invalid input, missing files. The difference between amateur and professional code is how errors are handled. JavaScript's `try...catch...finally` gives you the tools to gracefully handle failures and provide meaningful feedback.

## Key Concepts

**Exception**: An error that occurs during program execution, interrupting normal flow.

**try block**: Code that might throw an error.

**catch block**: Code that handles the error if one occurs.

**finally block**: Code that runs regardless of success or failure.

## Real World Context

API calls can fail, JSON parsing can throw, user input can be invalid. Production applications need robust error handling to provide good user experiences and meaningful logs for debugging.

## Deep Dive

### Basic Structure

```javascript
try {
  // Code that might fail
  const data = JSON.parse(userInput);
  processData(data);
} catch (error) {
  // Handle the error
  console.error('Failed to parse:', error.message);
  showUserError('Invalid input format');
} finally {
  // Always runs (cleanup)
  hideLoadingSpinner();
}
```

### The Error Object

```javascript
try {
  throw new Error('Something went wrong');
} catch (error) {
  console.log(error.name);     // 'Error'
  console.log(error.message);  // 'Something went wrong'
  console.log(error.stack);    // Stack trace
}
```

### Optional Catch Binding (ES2019)

```javascript
// When you don't need the error object
try {
  riskyOperation();
} catch {
  // No error parameter needed
  console.log('Operation failed');
}
```

### Nested Try-Catch

```javascript
try {
  try {
    throw new Error('Inner error');
  } finally {
    console.log('Inner finally');  // Runs
  }
} catch (error) {
  console.log('Outer catch:', error.message);
}
// Output:
// 'Inner finally'
// 'Outer catch: Inner error'
```

### Finally Always Runs

```javascript
function getData() {
  try {
    return fetchData();  // Even with return...
  } finally {
    cleanup();  // ...finally still runs!
  }
}

// finally runs even when no error
try {
  console.log('Success!');
} catch (e) {
  console.log('Error');  // Doesn't run
} finally {
  console.log('Cleanup');  // Always runs
}
```

### Re-throwing Errors

```javascript
try {
  processData(data);
} catch (error) {
  if (error instanceof ValidationError) {
    showValidationMessage(error.message);
  } else {
    // Re-throw unexpected errors
    throw error;
  }
}
```

## Common Pitfalls

1. **Catching and ignoring errors**: `catch (e) {}` hides bugs.
2. **Not using finally for cleanup**: Resources may not be released.
3. **Catching too broadly**: Handle specific errors, re-throw others.

## Best Practices

- **Always log or handle errors**: Never silently ignore them.
- **Use finally for cleanup**: Closing connections, hiding loaders.
- **Provide user-friendly messages**: Don't expose technical details.
- **Re-throw unexpected errors**: Don't swallow errors you can't handle.

## Summary

`try` wraps code that might throw. `catch` handles errors. `finally` always runs for cleanup. The error object contains `name`, `message`, and `stack`. Always handle or log errors—never ignore them.

## Code Examples

**Basic Structure**

```javascript
try {
  // Code that might fail
  const data = JSON.parse(userInput);
  processData(data);
} catch (error) {
  // Handle the error
  console.error('Failed to parse:', error.message);
  showUserError('Invalid input format');
} finally {
  // Always runs (cleanup)
  hideLoadingSpinner();
}
```

**The Error Object**

```javascript
try {
  throw new Error('Something went wrong');
} catch (error) {
  console.log(error.name);     // 'Error'
  console.log(error.message);  // 'Something went wrong'
  console.log(error.stack);    // Stack trace
}
```


## Resources

- [MDN: try...catch](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch) — Complete try-catch reference
- [MDN: Error](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error) — Error object reference

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*