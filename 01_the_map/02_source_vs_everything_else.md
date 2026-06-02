# 01.2 — Source vs Tests vs Config vs Build vs Meta

> Goal: cement the 5-bucket instinct so you never confuse "the program" with "stuff around the program."

---

## Why this distinction is the whole game

Beginners get overwhelmed because they treat all 40 files as equally important. They aren't. **Only bucket ① (source) is the program.** The other four exist to *develop, test, configure, ship, and explain* the program. Once you bucket files automatically, a scary repo becomes simple.

---

## ① Source code — the program itself
- **What:** the `.py` files containing the real logic.
- **Where:** `src/<pkg>/`, or `<pkg>/`, or `libs/<name>/<name>/`.
- **How to spot it:** it's what gets *imported* and *run*. It's inside the "package" (folder with `__init__.py`).
- **Your work:** every bug you fixed was here — e.g. `libs/agno/agno/integrations/discord/client.py`.

## ② Tests — code that checks the code
- **What:** `.py` files that import the source and `assert` it behaves correctly.
- **Where:** `tests/` (mirrors the source structure), or `*_test.py` next to the source.
- **How to spot it:** functions named `test_*`; imports `pytest`.
- **Key:** tests are NOT shipped to users. They're for developers/CI. You *added* tests to most PRs (e.g. `tests/litellm/caching/test_background_cache_tasks.py`).

## ③ Config — settings & secrets
- **What:** values that change *how* the program runs without changing its code: API keys, database URLs, feature flags, tool settings.
- **Where:** `.env` (secrets, local, gitignored), `pyproject.toml` (`[tool.*]` sections), `*.yaml`/`*.toml`/`*.json`/`*.ini`.
- **Rule:** config is *data*, not logic. Same code, different config = different behavior (dev vs production).

## ④ Dependencies & build — other people's code + how to package yours
- **What:** declarations of which external libraries you need, and recipes to bundle your app.
- **Where:** `requirements.txt`, `uv.lock`, `poetry.lock`, `Dockerfile`, `docker-compose.yml`.
- **Why separate:** you don't commit the actual downloaded libraries (that's `.venv/`/`node_modules/`, gitignored) — only the *list* of what to install.

## ⑤ Project meta — docs, license, CI, git
- **What:** everything that supports the project but isn't code/test/config/deps.
- **Where:** `README.md`, `LICENSE`, `CONTRIBUTING.md`, `.github/`, `.gitignore`, `Makefile`, `docs/`.
- **Why it matters:** `CONTRIBUTING.md` and `.github/` tell you the *rules* — you read these before contributing (e.g. agno required a `Fixes #123` issue link; vllm bans tiny PRs).

---

## A quick drill — bucket these at a glance

| File | Bucket | Why |
|------|--------|-----|
| `src/codehound/cli.py` | ① source | real logic, gets run |
| `tests/test_checks.py` | ② tests | `test_*`, verifies source |
| `.env` | ③ config | secrets/local settings |
| `pyproject.toml` | ③ + ④ | tool config AND dependency declaration |
| `uv.lock` | ④ deps | exact pinned versions |
| `Dockerfile` | ④ build | packaging recipe |
| `.github/workflows/ci.yml` | ⑤ meta | CI automation |
| `README.md` | ⑤ meta | docs |
| `__pycache__/` | (ignore) | generated cache |

Do this drill on any repo you open. In a week it's automatic.

---

## The "where do I start reading a new repo?" recipe

1. **`README.md`** — what it is, how to run it. (2 min)
2. **The source package** (`src/<pkg>/` etc.) — skim `__init__.py` to see what's exported, then the main modules. (the meat)
3. **`tests/`** — tests are often the *clearest documentation* of how code is meant to be used.
4. **`pyproject.toml`** — what it depends on, what tools it uses.
5. **`CONTRIBUTING.md` + `.github/`** — the rules, if you plan to contribute.

That order takes you from "what is this?" to "I could change this" fast.

---

## ✅ Say this in an interview

> *"I think of any repo as five buckets: source code, tests, config, dependencies/build, and project meta. Only the source is the actual program — the rest develops, tests, configures, ships, and documents it. When I open a new repo I read the README, skim the source package's `__init__` to see what's exported, then read the tests since they double as usage docs, and finally `CONTRIBUTING.md` for the rules. That's how I got productive fast in repos like agno and litellm."*

Next section: Python packaging — what makes code importable.
