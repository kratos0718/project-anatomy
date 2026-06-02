# 01.1 — Anatomy of a Project (the annotated map)

> Goal: open ANY repo and instantly know what each top-level file/folder is. This is the master map; later lessons zoom into each piece.

---

## A typical Python project, fully annotated

Here's what a well-organized Python library looks like when you `ls` it. **Read every comment** — this single tree is 60% of the whole repo's value:

```
my-project/
│
├── src/                      # ① SOURCE CODE — the actual program lives here
│   └── my_project/           #    the "package" (importable as `import my_project`)
│       ├── __init__.py       #    marks this folder as a package; runs on import
│       ├── __main__.py       #    lets you run `python -m my_project`
│       ├── core.py           #    a module (just a .py file with code)
│       ├── cli.py            #    another module (command-line interface)
│       └── checks/           #    a sub-package (folder with its own __init__.py)
│           ├── __init__.py
│           └── rules.py
│
├── tests/                    # ② TESTS — code that verifies the source code
│   ├── __init__.py
│   ├── conftest.py           #    shared pytest setup/fixtures
│   └── test_core.py          #    a test file (functions named test_*)
│
├── docs/                     # ⑤ DOCUMENTATION (sometimes its own site)
│
├── .github/                  # ⑤ GitHub-specific automation & templates
│   ├── workflows/
│   │   └── ci.yml            #    CI: runs tests/lint on every push (GitHub Actions)
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── ISSUE_TEMPLATE/
│
├── pyproject.toml            # ③ THE central config: package metadata, deps, tool settings
├── requirements.txt          # ④ list of dependencies to install (older style)
├── uv.lock / poetry.lock     # ④ exact pinned versions (lockfile) for reproducibility
│
├── .env                      # ③ SECRETS & local config (NEVER committed — gitignored)
├── .env.example              # ③ a safe template showing which env vars are needed
├── .gitignore                # ⑤ list of files git should IGNORE (never upload)
├── .pre-commit-config.yaml   # ⑤ hooks that auto-run lint/format before each commit
│
├── Dockerfile                # ④ recipe to package the app into a container image
├── docker-compose.yml        # ④ defines multi-service local setup (app + db + redis)
├── Makefile                  # ⑤ shortcuts: `make test`, `make lint`, `make format`
│
├── README.md                 # ⑤ the front page: what it is, how to install/use
├── LICENSE                   # ⑤ legal terms (MIT, Apache-2.0, etc.)
└── CONTRIBUTING.md           # ⑤ how to contribute (rules you must follow)
```

The circled numbers map to the **5 kinds** from the README:
① source · ② tests · ③ config · ④ dependencies/build · ⑤ project meta.

---

## The "I only care about the real code, where is it?" answer

When you clone a repo and want the actual logic, look for **one of these** (in order of likelihood):
1. `src/<package_name>/` — modern "src layout" (codehound uses this)
2. `<package_name>/` directly at the root — older flat layout
3. For monorepos: `libs/<name>/<name>/` or `packages/<name>/src/` (agno, litellm, chainlit do this)

Everything *else* in the tree is support: tests, config, docs, CI. The real program is usually <40% of the files.

---

## Files you can almost always ignore (as a contributor)

These are auto-generated or tooling noise — don't touch, don't fear:
- `__pycache__/`, `*.pyc` — Python's compiled cache. Auto-made. Gitignored.
- `.venv/`, `venv/` — your local virtual environment (lesson 04.1). Gitignored.
- `*.egg-info/`, `dist/`, `build/` — packaging output (lesson 06.2). Gitignored.
- `.mypy_cache/`, `.pytest_cache/`, `.ruff_cache/` — tool caches. Gitignored.
- `node_modules/` — JS dependencies (if there's a frontend). Huge, gitignored.

**Pattern:** if a folder is in `.gitignore`, it's generated/local — not something you edit. (Lesson 03.2 covers `.gitignore`.)

---

## How this maps to repos you actually contributed to

| Repo | Where the real code is | Notable |
|------|------------------------|---------|
| **codehound** (yours) | `src/codehound/` | clean `src/` layout, `checks/` sub-package |
| **agno** | `libs/agno/agno/` | monorepo — package nested two deep |
| **litellm** | `litellm/` (flat) + `litellm/proxy/` | huge flat package + a sub-app |
| **marimo** | `marimo/` (Python) + `frontend/` (React) | full-stack: Python backend + JS frontend |

You navigated all of these. Now you know *why* the code was where it was.

---

## ✅ Say this in an interview

> *"I can read any repo's layout quickly: source usually lives in `src/<package>/` or a nested `libs/<name>/`, tests in `tests/`, config in `pyproject.toml` and `.env`, dependencies in requirements or a lockfile, and project meta like CI in `.github/`. Anything in `.gitignore` — caches, virtualenvs, build output — is generated and not edited. For example agno is a monorepo with the package at `libs/agno/agno/`, while my own tool codehound uses a clean `src/` layout."*

Next: the source-vs-everything-else distinction in depth.
