---
source_course: "python"
source_lesson: "python-pathlib"
---

# Pathlib: Modern Path Handling

`pathlib` provides an object-oriented interface for filesystem paths.

## Creating Paths

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

## Path Properties

```python
p = Path('/home/user/docs/report.pdf')

p.name        # 'report.pdf'
p.stem        # 'report'
p.suffix      # '.pdf'
p.parent      # Path('/home/user/docs')
p.parts       # ('/', 'home', 'user', 'docs', 'report.pdf')
p.is_absolute()  # True
```

## File Operations

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

## Directory Operations

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


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*