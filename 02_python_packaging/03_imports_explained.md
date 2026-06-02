# 02.3 — Imports: absolute vs relative, PYTHONPATH, the `src/` layout

> Goal: never be confused by `from ..core import x` or `PYTHONPATH=src` again.

---

## Absolute imports — the full path from the package root

```python
from codehound.core import scan_path
import codehound.checks.blocking_async
```
You name the package from its top. Clear, explicit, the recommended default. Works anywhere as long as the package is installed or findable.

## Relative imports — "relative to where I am"

Inside a package, you can import siblings using dots:
```python
from .core import scan_path        # . = "this package" (same folder)
from ..utils import helper         # .. = "parent package" (one folder up)
```
- `.` = current package
- `..` = parent package

Example from codehound's checks: `from codehound.core import Check, Finding` (absolute) — they chose absolute for clarity. Many repos use relative (`from ..agent import Agent`) for brevity. Both are fine; some projects' style guides pick one (litellm's rule: "avoid double-dot relative imports, prefer absolute").

> Rule of thumb: relative imports only work *inside* a package (a folder with `__init__.py`), and only when the file is imported as part of the package — not when run directly as a loose script. That trips people up.

---

## How Python finds a package to import (`sys.path`)

When you write `import codehound`, Python looks through a list of directories called **`sys.path`**, in order:
1. The directory of the script you ran (or current dir).
2. Directories listed in the **`PYTHONPATH`** environment variable.
3. The installed-packages folders (`site-packages`) — where `pip install` puts things.

If `codehound` isn't in any of those, you get `ModuleNotFoundError: No module named 'codehound'`.

## `PYTHONPATH` — "also look here for packages"

`PYTHONPATH` is an environment variable holding extra folders to search.
```bash
PYTHONPATH=src python -m codehound.cli scan .
```
This says "also look inside `src/` for importable packages, then run codehound." You used this exact command when codehound wasn't pip-installed — it let Python find `src/codehound/` without installing it.

---

## The `src/` layout (why code is in `src/<pkg>/`)

Two common layouts:

**Flat layout** (older):
```
my-project/
├── my_package/        ← package at the root
└── tests/
```

**src layout** (modern, recommended — codehound uses it):
```
my-project/
├── src/
│   └── my_package/    ← package one level down, inside src/
└── tests/
```

**Why bother with `src/`?** It prevents a subtle bug: with the flat layout, Python might accidentally import your package from the project folder *before* it's properly installed, hiding packaging mistakes. The `src/` layout forces you to install the package to import it, so your tests run against the *installed* version — exactly how users will get it. It's a "test it like it ships" safety measure.

---

## The `__init__.py`-controls-the-API recap

Combine with lesson 02.1: a package's `__init__.py` decides what `from package import X` exposes. So:
- **Internal modules**: `package/core.py`, `package/_helpers.py` (the `_` prefix hints "private/internal").
- **Public API**: re-exported in `__init__.py` so users import cleanly.

This is why big libraries have a clean `from agno import Agent` even though `Agent` actually lives deep in `agno/agent/agent.py` — the top `__init__.py` re-exports it.

---

## ✅ Say this in an interview

> *"Absolute imports name the package from its root; relative imports use dots — `.` for the current package, `..` for the parent — and only work inside a package. Python finds packages via `sys.path`: the current dir, `PYTHONPATH`, then installed site-packages. The `src/` layout puts the package under `src/` so it can only be imported after installation, which makes tests run against the shipped version and catches packaging bugs. I used `PYTHONPATH=src` to run codehound before installing it, and a package's `__init__.py` is what re-exports the clean public API."*

Next section: the `.env` and config question you specifically asked about.
