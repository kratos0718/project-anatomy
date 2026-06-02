# 🏗️ Project Anatomy — What's Actually Inside a Real Codebase

**The problem this solves:** you open a real project (like agno or litellm) and see 40 files and folders — `pyproject.toml`, `.env`, `__init__.py`, `Dockerfile`, `.github/`, `tests/`, `requirements.txt`... and have no idea what any of them do. This repo explains **every single one**, from zero, so when you `git clone` anything, you instantly understand the layout.

**Companion to** [oss-learning-path](https://github.com/kratos0718/oss-learning-path) (that one explains *concepts* like async; this one explains *files and structure*).

---

## Read in this order

### 01 — The Map (start here)
- [`01_the_map/01_anatomy_of_a_project.md`](01_the_map/01_anatomy_of_a_project.md) — a full annotated folder tree; what every top-level file/folder is
- [`01_the_map/02_source_vs_everything_else.md`](01_the_map/02_source_vs_everything_else.md) — "src code" vs config vs tests vs docs vs CI

### 02 — Python packaging (how code becomes importable)
- [`02_python_packaging/01_modules_packages_init.md`](02_python_packaging/01_modules_packages_init.md) — `.py`, package, `__init__.py`, `__main__.py`
- [`02_python_packaging/02_pyproject_setup.md`](02_python_packaging/02_pyproject_setup.md) — `pyproject.toml`, `setup.py`, what "installing a package" means
- [`02_python_packaging/03_imports_explained.md`](02_python_packaging/03_imports_explained.md) — absolute vs relative imports, `PYTHONPATH`, `src/` layout

### 03 — Config & environment (the `.env` question)
- [`03_config_and_env/01_env_and_dotenv.md`](03_config_and_env/01_env_and_dotenv.md) — environment variables, `.env`, secrets, why they're gitignored
- [`03_config_and_env/02_config_files.md`](03_config_and_env/02_config_files.md) — `.gitignore`, YAML/TOML/JSON/INI config, `settings.py`

### 04 — Dependencies (other people's code)
- [`04_dependencies/01_pip_and_requirements.md`](04_dependencies/01_pip_and_requirements.md) — `pip`, `requirements.txt`, virtual environments, `.venv`
- [`04_dependencies/02_modern_tools.md`](04_dependencies/02_modern_tools.md) — `uv`, `poetry`, lockfiles, `package.json` (the JS side)

### 05 — Quality & CI (the robots)
- [`05_quality_and_ci/01_tests_lint_format.md`](05_quality_and_ci/01_tests_lint_format.md) — `tests/`, `conftest.py`, ruff/black/mypy config, `Makefile`, `.pre-commit-config.yaml`
- [`05_quality_and_ci/02_github_folder.md`](05_quality_and_ci/02_github_folder.md) — the whole `.github/` folder: workflows, PR templates, issue templates

### 06 — Deployment & packaging artifacts
- [`06_deployment/01_docker_and_containers.md`](06_deployment/01_docker_and_containers.md) — `Dockerfile`, `docker-compose.yml`, images vs containers
- [`06_deployment/02_build_artifacts.md`](06_deployment/02_build_artifacts.md) — `dist/`, `build/`, `.egg-info`, wheels, what gets published

### 07 — Real walkthroughs (apply it)
- [`07_walkthroughs/01_a_small_library.md`](07_walkthroughs/01_a_small_library.md) — codehound's layout, file by file
- [`07_walkthroughs/02_a_big_framework.md`](07_walkthroughs/02_a_big_framework.md) — agno / litellm: how huge monorepos are organized
- [`07_walkthroughs/03_a_web_app.md`](07_walkthroughs/03_a_web_app.md) — a typical FastAPI + frontend app (backend/, frontend/, .env)

---

## The 30-second mental model

Every project, no matter how big, is just **5 kinds of things**:

| Kind | "Is this the actual program?" | Examples |
|------|-------------------------------|----------|
| **1. Source code** | YES — the real logic | `src/`, the main package folder |
| **2. Tests** | No — code that checks the code | `tests/`, `*_test.py` |
| **3. Config** | No — settings & secrets | `.env`, `pyproject.toml`, `*.yaml` |
| **4. Dependency/build info** | No — what to install & how to package | `requirements.txt`, `uv.lock`, `Dockerfile` |
| **5. Project meta** | No — docs, license, CI, git stuff | `README`, `LICENSE`, `.github/`, `.gitignore` |

Learn to glance at any file and bucket it into one of these 5. That's the whole skill. Every lesson here makes that instinct sharper.
