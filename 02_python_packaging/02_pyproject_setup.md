# 02.2 — `pyproject.toml`, `setup.py`, and "Installing a Package"

> Goal: understand the single most important config file in modern Python projects.

---

## What "installing a package" actually means

When you `pip install requests`, pip downloads the `requests` code and puts it in a special folder Python always searches (`site-packages`). After that, `import requests` works from anywhere.

A project needs to *describe itself* so it can be installed and shared: its name, version, what it depends on, where its code is. That description lives in **`pyproject.toml`**.

---

## `pyproject.toml` — the modern project file

`.toml` = "Tom's Obvious Minimal Language" — a simple, readable config format (key = value, grouped in `[sections]`). `pyproject.toml` is the **one file** that modern Python projects use for everything: packaging info AND tool settings.

Annotated example (close to codehound's real one):

```toml
[build-system]                       # how to BUILD this package
requires = ["hatchling"]             #   the build tool to use
build-backend = "hatchling.build"

[project]                            # the package's IDENTITY
name = "codehound"                   #   what you `pip install`
version = "0.1.0"                    #   its version
description = "AST static analyzer..."
requires-python = ">=3.9"            #   minimum Python version
dependencies = []                    #   external libs it needs (none here!)

[project.optional-dependencies]      # extra deps only some users need
dev = ["pytest>=7"]                  #   `pip install codehound[dev]` adds pytest

[project.scripts]                    # command-line entry points
codehound = "codehound.cli:main"     #   makes a `codehound` terminal command

[tool.pytest.ini_options]            # CONFIG FOR A TOOL (pytest) — note [tool.*]
testpaths = ["tests"]

[tool.ruff]                          # config for the ruff linter
line-length = 88
```

### The three things to notice
1. **`[project]`** — identity + `dependencies` (what to install alongside it).
2. **`[project.scripts]`** — this is the magic that turns `codehound.cli:main` into a real `codehound` terminal command after install.
3. **`[tool.xxx]` sections** — `pyproject.toml` also holds settings for tools like ruff, black, mypy, pytest. One file, everything. (This is why you edit `pyproject.toml` to change line length or test paths.)

---

## The old way: `setup.py` / `setup.cfg`

Before `pyproject.toml`, packaging info lived in **`setup.py`** (an actual Python script) and/or `setup.cfg`. You'll still see these in older repos. They do the same job — describe the package — just in an older format. If you see both `setup.py` and `pyproject.toml`, the project is mid-migration; `pyproject.toml` wins.

> Real memory: your first codehound install failed because old `pip` couldn't do an "editable install" from a hatchling-only `pyproject.toml` — that's exactly this old-vs-new tooling friction.

---

## "Editable install" — the dev superpower

```bash
pip install -e .
```
The `-e` ("editable") installs your package but **links** to your source folder instead of copying it. So you can edit the code and the changes take effect immediately, no reinstall. Every Python project you develop locally gets installed this way. The `.` means "the package in the current directory" (it reads `pyproject.toml`).

---

## What a "build" produces (preview of lesson 06.2)

When a package is "built" for publishing, the build tool (hatchling/setuptools/poetry) reads `pyproject.toml` and produces:
- a **wheel** (`.whl`) — a pre-built, ready-to-install zip,
- a **sdist** (`.tar.gz`) — the source distribution.

These go to **PyPI** (the Python Package Index, pypi.org) so others can `pip install` them. Your `huggingface_hub` fix shipped in the `v1.17.0` wheel on PyPI — that's why `pip install huggingface_hub==1.17.0` runs your code.

---

## ✅ Say this in an interview

> *"`pyproject.toml` is the modern single config file for a Python project — it holds the package identity and dependencies under `[project]`, command-line entry points under `[project.scripts]`, and tool settings like ruff/pytest under `[tool.*]`. Older projects use `setup.py` instead. During development you `pip install -e .` for an editable install that links to your source so edits apply instantly. When published, the build produces a wheel that goes to PyPI — which is how my huggingface_hub fix shipped in the v1.17.0 release."*

Next: how imports actually resolve (absolute vs relative, the src layout).
