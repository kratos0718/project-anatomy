# 05.1 — tests/, conftest.py, Makefile, pre-commit

> Goal: understand the developer-quality files you'll find in every serious repo.

(Concepts of testing/lint/CI are in the companion repo `oss-learning-path/05_tooling`. This lesson is about the **files & folders** they live in.)

---

## The `tests/` folder

Holds all the test code. It usually **mirrors the source structure**:
```
src/codehound/checks/blocking_async.py   ← source
tests/test_checks.py                     ← its test
```
Big projects mirror exactly:
```
litellm/caching/caching_handler.py          ← source
tests/litellm/caching/test_background_...py  ← your test went here
```
So to find a file's tests, look for the same path under `tests/`. To find what a file is *supposed* to do, **read its tests** — they're executable documentation.

- Files are named `test_*.py` (or `*_test.py`).
- Functions are named `test_*`.
- `tests/__init__.py` (often empty) marks it a package.

## `conftest.py` — shared pytest setup

A special pytest file (you don't import it; pytest finds it automatically). It holds **fixtures** — reusable setup shared across many test files. Example: a fixture that creates a temporary database, or a fake user. If you see `conftest.py`, it's "shared test plumbing," not actual tests.

---

## Where tool settings live

Linters/formatters/type-checkers read their settings from `pyproject.toml` (the `[tool.*]` sections) or dedicated files:

| File | Tool | Purpose |
|------|------|---------|
| `[tool.ruff]` in `pyproject.toml` | ruff | linter rules, line length |
| `[tool.black]` | black | formatter settings |
| `[tool.mypy]` or `mypy.ini` | mypy | type-checking strictness |
| `[tool.pytest.ini_options]` or `pytest.ini` | pytest | where tests are, options |
| `.flake8`, `tox.ini` | older tools | similar settings |

You edit these when CI complains about style. (You edited line-length/test-path settings in codehound's `pyproject.toml`.)

---

## `Makefile` — command shortcuts

A `Makefile` defines named shortcut commands so you don't memorize long ones:
```makefile
test:
	pytest tests/

lint:
	ruff check . && mypy .

format:
	black . && ruff check --fix .
```
Run them with `make test`, `make lint`, `make format`. Most big repos have a Makefile so every contributor runs checks the same way. (litellm's CLAUDE.md: `make lint`, `make format`, `make test-unit`. agno: `./scripts/format.sh`, `./scripts/validate.sh` — same idea via shell scripts instead of make.)

**When you join a repo, the Makefile (or `scripts/`) tells you the exact commands to validate your change before pushing** — run those and CI will pass.

---

## `.pre-commit-config.yaml` — auto-checks before each commit

**pre-commit** is a tool that runs checks **automatically every time you `git commit`** — before the commit is even created. Configured in `.pre-commit-config.yaml`:
```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    hooks:
      - id: ruff          # lint on every commit
      - id: ruff-format   # auto-format on every commit
```
Install once with `pre-commit install`. After that, committing runs ruff/black/etc.; if they fail or auto-fix files, the commit stops so you can review. This catches style issues *before* they ever reach CI. (You saw `pre-commit-ci[bot]` push auto-fix commits on marimo's PR — that's this system running in the cloud.)

---

## `scripts/` folder

Many repos have a `scripts/` folder with helper shell/Python scripts: `scripts/format.sh`, `scripts/setup.sh`, `scripts/run_tests.sh`. These are convenience automation — read them to learn how the project wants things done.

---

## ✅ Say this in an interview

> *"Tests live in `tests/`, usually mirroring the source path, with `conftest.py` holding shared pytest fixtures. Tool settings live in `pyproject.toml`'s `[tool.*]` sections or files like `mypy.ini`. A `Makefile` (or a `scripts/` folder) gives shortcut commands like `make lint`, `make test`, `make format` — the exact checks I run before pushing so CI passes. And `.pre-commit-config.yaml` runs lint/format automatically on every commit, catching issues before they reach CI. When I join a repo I check the Makefile to learn its validation commands."*

Next: the whole `.github/` folder.
