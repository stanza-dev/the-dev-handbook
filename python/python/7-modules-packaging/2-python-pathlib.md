---
source_course: "python"
source_lesson: "python-pathlib"
---

# Modern File Paths with Pathlib

## Introduction
The `pathlib` module provides an object-oriented, cross-platform way to work with filesystem paths. It replaces the fragmented `os.path` API with a single, intuitive `Path` object that handles joining, reading, writing, and globbing.

## Key Concepts
- **`Path`**: An object representing a filesystem path, with methods for reading, writing, and querying.
- **`/` operator**: Joins path segments -- `Path('data') / 'file.txt'` produces `data/file.txt`.
- **Path properties**: `.name`, `.stem`, `.suffix`, `.parent`, `.parts` -- extract components without string parsing.
- **`glob()` / `rglob()`**: Find files matching a pattern in a directory or recursively.

## Real World Context
Every Python application that touches the filesystem benefits from pathlib. Configuration loaders, test fixtures, static file servers, and build scripts all need to join paths, check existence, and read files. Pathlib's cross-platform handling means your code works on Windows, macOS, and Linux without `os.sep` gymnastics.

## Deep Dive

### Creating Paths

```python
from pathlib import Path

# Current directory
cwd = Path.cwd()

# Home directory
home = Path.home()

# From string
p = Path('/usr/local/bin')
p = Path('data/file.txt')

# Joining paths (use / operator)
path = Path('data') / 'subdir' / 'file.txt'
```

### Path Properties

```python
p = Path('/home/user/docs/report.pdf')

p.name        # 'report.pdf'
p.stem        # 'report'
p.suffix      # '.pdf'
p.parent      # Path('/home/user/docs')
p.parts       # ('/', 'home', 'user', 'docs', 'report.pdf')
p.is_absolute()  # True
```

### File Operations

```python
p = Path('data.txt')

# Read/Write
content = p.read_text()           # Read entire file
p.write_text('Hello World')      # Write string
bytes_data = p.read_bytes()       # Read binary
p.write_bytes(b'\x00\x01')       # Write binary

# Check existence
p.exists()     # True/False
p.is_file()    # Is it a file?
p.is_dir()     # Is it a directory?

# Create/Delete
p.touch()      # Create empty file
p.mkdir(parents=True, exist_ok=True)  # Create directory
p.unlink()     # Delete file
p.rmdir()      # Delete empty directory
```

### Directory Operations

```python
dir_path = Path('project')

# List contents
for item in dir_path.iterdir():
    print(item)

# Glob patterns
for py_file in dir_path.glob('*.py'):
    print(py_file)

# Recursive glob
for py_file in dir_path.rglob('*.py'):
    print(py_file)  # All .py files in subdirectories
```

## Common Pitfalls
1. **Mixing `os.path` strings with `Path` objects** -- Functions like `os.path.join()` expect strings. Either commit to pathlib throughout or convert with `str(path)` at boundaries.
2. **Using `write_text()` without considering encoding** -- By default it uses the system encoding. Pass `encoding='utf-8'` explicitly to avoid cross-platform surprises.
3. **Forgetting `parents=True` in `mkdir()`** -- Without it, creating a nested directory like `a/b/c` fails if `a/b` does not exist.

## Best Practices
1. **Replace all `os.path` usage with `pathlib`** -- `Path(base) / 'sub' / 'file.txt'` is more readable than `os.path.join(base, 'sub', 'file.txt')`.
2. **Use `rglob()` for recursive file discovery** -- It is cleaner than `os.walk()` and returns `Path` objects directly.

## Summary
- `pathlib.Path` provides an object-oriented, cross-platform API for filesystem paths.
- The `/` operator joins paths cleanly; `.name`, `.stem`, `.suffix` extract components.
- `read_text()`, `write_text()`, `mkdir()`, `glob()`, and `rglob()` cover most file operations.
- Always use `parents=True, exist_ok=True` with `mkdir()` for robust directory creation.
- Prefer pathlib over `os.path` for all new Python code.

## Code Examples

**Practical pathlib usage**

```python
from pathlib import Path

# Common patterns
config_dir = Path.home() / '.config' / 'myapp'
config_dir.mkdir(parents=True, exist_ok=True)

config_file = config_dir / 'settings.json'
if not config_file.exists():
    config_file.write_text('{}')

# Replace os.path patterns
# Old: os.path.join(base, 'sub', 'file.txt')
# New: Path(base) / 'sub' / 'file.txt'

# Old: os.path.splitext(filename)[0]
# New: Path(filename).stem
```


## Resources

- [pathlib Module](https://docs.python.org/3.14/library/pathlib.html) — Official Python 3.14 reference for object-oriented filesystem paths

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*