---
source_course: "django-foundations"
source_lesson: "django-foundations-setting-up-environment"
---

# Setting Up Your Django Environment

Before creating Django applications, you need to install Python and set up a proper development environment. Let's get everything ready.

## Prerequisites

Django 6.0 requires:
- **Python 3.12, 3.13, or 3.14** (we recommend the latest stable release)
- **pip** (Python package installer)
- A code editor (VS Code, PyCharm, etc.)

## Step 1: Install Python

### On macOS

Using Homebrew (recommended):

```bash
# Install Homebrew if not already installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Python
brew install python@3.13

# Verify installation
python3 --version
# Python 3.13.x
```

### On Ubuntu/Debian

On Ubuntu or Debian-based systems, Python is available through the system package manager.

```bash
# Update package list
sudo apt update

# Install Python and pip
sudo apt install python3 python3-pip python3-venv

# Verify installation
python3 --version
```

### On Windows

1. Download Python from https://www.python.org/downloads/
2. Run the installer
3. **Important**: Check "Add Python to PATH"
4. Open Command Prompt and verify:

```batch
python --version
```

## Step 2: Create a Virtual Environment

Always use a virtual environment to isolate your project dependencies:

```bash
# Create a directory for your project
mkdir myproject
cd myproject

# Create a virtual environment
python3 -m venv venv

# Activate the virtual environment
# On macOS/Linux:
source venv/bin/activate

# On Windows:
venv\Scripts\activate
```

Your terminal prompt should now show `(venv)` indicating the virtual environment is active.

## Step 3: Install Django

With your virtual environment activated:

```bash
# Install Django
pip install django

# Verify installation
python -m django --version
# 6.0.x
```

## Step 4: Verify Everything Works

Let's confirm that Django is installed correctly and accessible from Python.

```bash
# Start a Python shell
python

>>> import django
>>> django.VERSION
(6, 0, 0, 'final', 0)
>>> exit()
```

## Recommended Code Editor Setup

We recommend **VS Code** with these extensions:

- **Python** - Official Python support
- **Pylance** - Fast language server
- **Django** - Django template support

## Project Structure Best Practice

Organize your Django projects like this:

```
projects/
├── project1/
│   ├── venv/           # Virtual environment
│   ├── mysite/         # Django project
│   └── requirements.txt
├── project2/
│   ├── venv/
│   └── ...
```

## Troubleshooting Common Issues

### "python: command not found"
Use `python3` instead of `python`, or add Python to your PATH.

### "Permission denied" errors
Never use `sudo pip install`. Always use a virtual environment.

### "No module named django"
Make sure your virtual environment is activated (`source venv/bin/activate`).

## Common Pitfalls

- **Installing Django globally with sudo**: Never use `sudo pip install django`. Always use a virtual environment to avoid permission issues and dependency conflicts.
- **Forgetting to activate the virtual environment**: If your terminal prompt does not show `(venv)`, Django commands will fail with "No module named django".
- **Using an unsupported Python version**: Django 6.0 requires Python 3.12 or later. Check your version with `python3 --version` before installing.

## Best Practices

- **Always use virtual environments**: Create a new virtual environment for each project to isolate dependencies.
- **Pin your Django version**: Use `pip install django==6.0.*` to avoid unexpected upgrades.
- **Create a requirements.txt early**: Run `pip freeze > requirements.txt` after installing packages so others can replicate your environment.

## Summary

- Django 6.0 requires **Python 3.12, 3.13, or 3.14**
- Always use a **virtual environment** (`python3 -m venv venv`) to isolate project dependencies
- Install Django with `pip install django` inside the activated virtual environment
- Verify installation with `python -m django --version`
- Use VS Code with the Python and Django extensions for the best development experience

## Code Examples

**Setting up an isolated Python virtual environment and installing Django 6.0**

```bash
# Create and activate a virtual environment, then install Django
python3 -m venv venv
source venv/bin/activate
pip install django
python -m django --version
```


## Resources

- [Django Installation Guide](https://docs.djangoproject.com/en/6.0/topics/install/) — Official guide to installing Django on all platforms

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*