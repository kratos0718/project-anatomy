# 04.2 — Modern Tools: uv, poetry, lockfiles, and the JS side

> Goal: recognize `uv.lock`, `poetry.lock`, `package.json`, `node_modules/` and know what they're for.

---

## The problem lockfiles solve: "works on my machine"

`requirements.txt` might say `requests>=2.28`. But `>=2.28` could install 2.28 today and 2.31 next month — and 2.31 might subtly break things. Two developers running the same `requirements.txt` weeks apart can get *different* versions → bugs that appear for one person and not another.

A **lockfile** fixes this: it records the **exact** version of *every* library, including dependencies-of-dependencies, so everyone installs an *identical* environment. Reproducibility.

```
requirements.txt :  "requests>=2.28"          ← a request (flexible)
lockfile         :  "requests==2.31.0 (sha256:...)" ← the exact resolved truth
```

---

## uv — the fast modern tool (you saw this a lot)

**`uv`** is a very fast, modern Python package manager + venv tool (written in Rust). It reads `pyproject.toml`, resolves dependencies, writes a **`uv.lock`** lockfile, and manages the venv — all much faster than pip.

```bash
uv venv                 # create a virtual environment
uv sync                 # install everything from pyproject.toml + uv.lock
uv run pytest           # run a command inside the managed env
uv pip install requests # pip-compatible install
```

agno, marimo, ragas, and vllm all used `uv` (you saw `uv run`, `.venv` references in their CLAUDE.md files). When a repo has a `uv.lock`, use `uv` commands.

## poetry — another popular manager

**`poetry`** does the same job (deps + venv + build), with its own `poetry.lock`. Commands like `poetry install`, `poetry add requests`, `poetry run pytest`. If you see `poetry.lock` and `[tool.poetry]` in `pyproject.toml`, the project uses poetry. (AutoGPT's backend used poetry.)

> **All three (pip, uv, poetry) do the same fundamental thing** — declare, resolve, and install dependencies into an isolated env. They differ in speed and ergonomics. Recognize the lockfile to know which one: `uv.lock` → uv, `poetry.lock` → poetry, neither → probably plain pip + requirements.txt.

---

## The JavaScript side (for full-stack repos like marimo)

If a project has a **frontend** (web UI), it has a *separate* dependency system for JavaScript:

### `package.json` — the JS equivalent of pyproject.toml
```json
{
  "name": "marimo-frontend",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "test": "vitest"
  },
  "dependencies": { "react": "^18.0.0" },
  "devDependencies": { "vitest": "^1.0.0" }
}
```
- `dependencies` / `devDependencies` — JS libraries (runtime vs dev-only).
- `scripts` — named commands you run with `npm run <name>` / `pnpm <name>` (e.g. `pnpm build`).

### Package managers: `npm`, `pnpm`, `yarn`
Same role as pip but for JS. `pnpm install` downloads dependencies into **`node_modules/`** (the JS equivalent of `.venv/` — huge, gitignored, recreated from `package.json`).

### Lockfiles: `package-lock.json` / `pnpm-lock.yaml` / `yarn.lock`
Same idea as `uv.lock` — exact versions for reproducibility.

> marimo (your feature PR) is full-stack: `marimo/` (Python backend) + `frontend/` (React, with `package.json` + `pnpm`). Your fix was in the Python part, but knowing the frontend exists explains the extra files.

---

## Quick recognition table

| You see... | It means... |
|------------|-------------|
| `requirements.txt` | pip-style dependency list |
| `pyproject.toml` + `uv.lock` | uv-managed project |
| `pyproject.toml` + `poetry.lock` | poetry-managed project |
| `.venv/` | Python virtual environment (gitignored) |
| `package.json` | JavaScript/frontend project |
| `node_modules/` | installed JS deps (gitignored, huge) |
| `*.lock` (any) | exact pinned versions for reproducibility |

---

## ✅ Say this in an interview

> *"pip, uv, and poetry all do the same job — resolve and install dependencies into an isolated environment — and each writes a lockfile (uv.lock, poetry.lock) that pins exact versions so everyone gets an identical, reproducible environment. uv is the fast modern one; I saw it across agno, marimo, and ragas. Full-stack repos like marimo also have a JS side: package.json declares frontend deps, pnpm installs them into node_modules, and there's a JS lockfile too. Recognizing the lockfile tells me which tool the project uses."*

Next section: tests, linting, the Makefile, and pre-commit.
