# 04.1 — pip, requirements.txt, and Virtual Environments

> Goal: understand how a project declares and installs the other libraries it depends on.

---

## Dependencies = other people's code your project needs

Almost no project is built from scratch. agno uses `requests`, `pydantic`, `openai`; your codehound uses only the standard library (a deliberate flex — "zero dependencies"). These external libraries are **dependencies**.

You don't copy their code into your repo. Instead you **declare** which ones you need, and a tool **installs** them. Two questions: *what to install* (a list) and *where to put it* (a virtual environment).

---

## pip — the installer

**pip** is Python's package installer. It downloads libraries from **PyPI** (pypi.org, the Python Package Index) and installs them so you can `import` them.
```bash
pip install requests          # install one library
pip install requests==2.31.0  # install an exact version
pip uninstall requests        # remove it
pip list                      # show what's installed
```

---

## `requirements.txt` — the dependency list (classic style)

A plain text file listing what to install, one per line:
```
# requirements.txt
requests>=2.28
pydantic==2.5.0
numpy
openai>=1.0,<2.0
```
- `requests>=2.28` — at least 2.28.
- `pydantic==2.5.0` — exactly that.
- `numpy` — any version.
- `openai>=1.0,<2.0` — a range.

Install everything at once:
```bash
pip install -r requirements.txt    # -r = "read this requirements file"
```

Projects often split it: `requirements.txt` (to run), `requirements-dev.txt` or `requirements/test.txt` (extra tools for developers: pytest, ruff). litellm had `requirements/test/cuda.txt` etc.

> Modern projects increasingly put dependencies in `pyproject.toml`'s `[project] dependencies` instead of `requirements.txt` (lesson 02.2). You'll see both.

---

## 🧪 Virtual environments — the "where to put it" answer

Here's a problem: Project A needs `numpy 1.20`, Project B needs `numpy 2.0`. If you install libraries globally (one shared place), they conflict. You can only have one `numpy`.

A **virtual environment (venv)** solves this: an *isolated* folder holding a private copy of Python + that project's own libraries. Each project gets its own. No conflicts.

```bash
python -m venv .venv          # create a venv in a folder called .venv
source .venv/bin/activate     # "enter" it (Mac/Linux); Windows: .venv\Scripts\activate
pip install -r requirements.txt   # now installs INTO .venv, isolated
# ... work ...
deactivate                    # "leave" the venv
```

- **`.venv/`** is the folder holding the isolated environment. It's **gitignored** (it's huge and machine-specific — you recreate it from `requirements.txt`, you don't share it).
- "Activating" makes your terminal use that venv's Python + libraries instead of the global one.

**Mental model:** `requirements.txt` is the *recipe* (what to install); `.venv/` is the *kitchen* (where it's installed, isolated per project). You share the recipe, not the kitchen.

> Real note: agno's CLAUDE.md mentioned two venvs — `.venv/` for development and `.venvs/demo/` for running cookbooks. Big projects sometimes have several for different purposes.

---

## The full setup flow for any cloned project

```bash
git clone <repo> && cd <repo>     # get the code
python -m venv .venv              # make an isolated env
source .venv/bin/activate         # enter it
pip install -r requirements.txt   # install its dependencies (the recipe)
# (or) pip install -e ".[dev]"    # editable install + dev extras from pyproject.toml
pytest                            # run its tests to confirm it works
```
This is the universal "I cloned a Python project, now make it run" sequence.

---

## ✅ Say this in an interview

> *"Dependencies are external libraries a project needs, declared in requirements.txt or pyproject.toml and installed by pip from PyPI. You install them into a virtual environment — an isolated per-project folder — so two projects can use different versions of the same library without conflict. The requirements file is the recipe you commit; the `.venv` folder is the isolated install, which is gitignored because it's recreated from the recipe. The standard flow for any repo is: clone, make a venv, activate, pip install the requirements, run the tests."*

Next: the modern tooling (uv, poetry, lockfiles) and the JS side (package.json).
