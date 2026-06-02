# 02.1 — Modules, Packages, `__init__.py`, `__main__.py`

> Goal: understand the folder/file structure of Python code itself — why `__init__.py` exists and what those `__double_underscore__` files do.

---

## Module = one `.py` file

A **module** is simply a single `.py` file. Its name is the filename without `.py`.
```python
# file: math_utils.py  →  module name: math_utils
def add(a, b): return a + b
```
Another file uses it:
```python
import math_utils
math_utils.add(2, 3)
```

## Package = a folder of modules

A **package** is a *folder* containing modules, so you can group related code:
```
codehound/
├── __init__.py        ← this file makes the folder a package
├── core.py            ← module codehound.core
├── cli.py             ← module codehound.cli
└── checks/            ← a SUB-package
    ├── __init__.py
    └── blocking_async.py   ← module codehound.checks.blocking_async
```
Now you can `import codehound.core` or `from codehound.checks.blocking_async import BlockingCallInAsync`. The dots mirror the folders.

---

## `__init__.py` — the file that confused you

**It marks a folder as a package**, and it **runs when the package is imported.** Two uses:

1. **Empty `__init__.py`** — just says "this folder is a package." (Common in `tests/`.)
2. **`__init__.py` with code** — defines what the package *exposes* at the top level. This is the "front door."

Example from codehound's `__init__.py`:
```python
from codehound.core import scan_path, Finding, Check
from codehound.checks import ALL_CHECKS
__version__ = "0.1.0"
```
This lets users do the clean `from codehound import scan_path` instead of the long `from codehound.core import scan_path`. **When you want to know "what does this package offer?", read its `__init__.py` first** — it's the curated public surface.

> Note: modern Python (3.3+) allows packages *without* `__init__.py` ("namespace packages"), but almost every real project still uses it, because it's where you control the public API.

---

## `__main__.py` — makes a package runnable

If a package has a `__main__.py`, you can run the whole package as a program:
```bash
python -m codehound          # runs codehound/__main__.py
```
The `-m` flag means "run this module/package as a script." It's how `python -m pytest`, `python -m pip`, `python -m http.server` all work.

---

## The `__dunder__` names (double underscore)

"Dunder" = "double underscore." These are special names Python recognizes:

| Name | Meaning |
|------|---------|
| `__init__.py` | package marker / package init code |
| `__main__.py` | entry point for `python -m package` |
| `__init__(self)` | a class's constructor method (different context, same naming idea) |
| `__version__` | convention for a package's version string |
| `__name__` | a module's name; `"__main__"` when run directly |
| `if __name__ == "__main__":` | "only run this block if this file is executed directly, not imported" |

That last one you'll see at the bottom of countless scripts:
```python
def main(): ...
if __name__ == "__main__":   # true only when you run THIS file directly
    main()
```
It lets a file be *both* importable (as a module) and runnable (as a script).

---

## How imports find packages

When you `import codehound`, Python searches a list of folders (`sys.path`): the current directory, installed-packages folders, and anything on `PYTHONPATH`. If your package is "installed" (lesson 02.2) or you set `PYTHONPATH=src`, Python can find it. This is why you sometimes ran `PYTHONPATH=src python -m codehound.cli` — telling Python "also look in `src/` for packages."

---

## ✅ Say this in an interview

> *"A module is a single .py file; a package is a folder of modules marked by an `__init__.py`, which also runs on import and defines the package's public surface. `__main__.py` makes a package runnable via `python -m`. The `if __name__ == '__main__'` guard lets a file be both imported and run directly. In my codehound tool the `checks/` sub-package groups one module per rule, and the top-level `__init__.py` re-exports the clean public API."*

Next: pyproject.toml and what "installing a package" means.
