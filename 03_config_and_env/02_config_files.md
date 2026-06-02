# 03.2 — `.gitignore` and the Config File Zoo (YAML/TOML/JSON/INI)

> Goal: recognize every config file format and know what `.gitignore` does.

---

## `.gitignore` — the "don't track this" list

A plain text file listing patterns of files/folders git should **completely ignore** — never track, never upload, never show in `git status`.

```gitignore
# .gitignore — real example
__pycache__/        # Python's compiled cache
*.pyc               # compiled python files
.venv/              # local virtual environment
.env                # SECRETS — critical to ignore
dist/               # build output
*.egg-info/         # packaging metadata
.pytest_cache/      # test tool cache
.DS_Store           # macOS folder junk
node_modules/       # JS dependencies (huge)
```

### Why ignore things?
1. **Secrets** — `.env` must never leak.
2. **Generated files** — caches, build output, compiled code: pointless to track (they're recreated automatically) and would bloat the repo.
3. **Local/machine-specific** — `.venv/`, `.DS_Store`: yours alone, irrelevant to others.

**The instinct from lesson 01.1 pays off:** *if it's in `.gitignore`, it's generated or local — not something you edit or worry about.*

Patterns: `*.pyc` = any file ending `.pyc`; `folder/` = that folder anywhere; `.env` = exactly that file.

---

## The config-format zoo (you'll see all of these)

Config = data that controls the program. It comes in several formats — same idea, different syntax:

### TOML (`.toml`) — modern Python config
```toml
[project]
name = "codehound"
line-length = 88
dependencies = ["requests", "numpy"]
```
Clean, typed, grouped in `[sections]`. Used by `pyproject.toml`. (Lesson 02.2.)

### YAML (`.yml` / `.yaml`) — indentation-based, very common in CI/DevOps
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pytest
```
Indentation = structure (like Python). Used by GitHub Actions (`.github/workflows/*.yml`), Docker Compose, Kubernetes. **Whitespace-sensitive** — a wrong indent breaks it.

### JSON (`.json`) — machine-friendly, used everywhere
```json
{
  "name": "my-frontend",
  "version": "1.0.0",
  "scripts": { "build": "vite build" }
}
```
Strict (double quotes, no comments, no trailing commas). Used by JS (`package.json`), API payloads, configs.

### INI / CONF (`.ini`, `.cfg`) — old-school
```ini
[settings]
debug = true
port = 8000
```
Simple sections + key=value. Seen in `setup.cfg`, `tox.ini`, some tool configs.

### `.env` — special-case (lesson 03.1)
Flat `KEY=value`, loaded into environment variables. For secrets/env config.

---

## How to tell config from code at a glance
- **Code** = `.py`, `.js`, `.ts` — has logic (functions, loops, `if`).
- **Config** = `.toml`, `.yaml`, `.json`, `.ini`, `.env` — has *data* (keys and values, no logic).

If you can run it, it's code. If it just *describes settings*, it's config.

---

## `settings.py` / `config.py` — config that IS code

Sometimes config lives in a Python file (e.g. Django's `settings.py`, or a `config.py` that reads env vars and exposes them):
```python
# config.py
import os
DEBUG = os.getenv("DEBUG", "false") == "true"
DATABASE_URL = os.getenv("DATABASE_URL")
OPENAI_KEY = os.getenv("OPENAI_API_KEY")
```
This is the bridge: it reads raw environment variables (from `.env`) and turns them into clean Python values the rest of the app imports. Common pattern: `from config import settings`.

---

## ✅ Say this in an interview

> *"`.gitignore` lists files git should never track — secrets like `.env`, plus generated stuff like `__pycache__`, `.venv`, and build output, because they're either sensitive or auto-recreated. Config comes in several formats: TOML for `pyproject.toml`, YAML for CI and Docker Compose, JSON for JS and APIs, and `.env` for environment variables. The distinction from code is simple — config is data describing settings, code has logic. A `config.py` often bridges them by reading env vars into typed Python settings the app imports."*

Next section: dependencies — pip, requirements, virtual environments.
