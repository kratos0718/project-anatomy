# 07.1 — Walkthrough: A Small Library (codehound, file by file)

> Goal: apply everything to a real, small, complete project — your own codehound. Once you can read this whole tree, you can read any small library.

---

## The full tree (real)

```
codehound/
├── src/
│   └── codehound/                 # ① the package (the actual tool)
│       ├── __init__.py            #    re-exports public API + __version__
│       ├── cli.py                 #    command-line interface (scan/list commands)
│       ├── core.py                #    engine: file discovery, AST parsing, parent map
│       └── checks/                #    sub-package: one module per rule
│           ├── __init__.py        #       registry of all checks (ALL_CHECKS)
│           ├── blocking_async.py  #       CH001
│           ├── mutable_defaults.py#       CH002
│           ├── datetime_utcnow.py #       CH003
│           ├── get_event_loop.py  #       CH004
│           ├── resource_leak.py   #       CH005
│           └── floating_task.py   #       CH006
├── tests/
│   └── test_checks.py             # ② tests for every check (good + bad cases)
├── docs/
│   ├── ARCHITECTURE.md            # ⑤ how the engine works
│   └── FINDINGS.md                # ⑤ provenance of each rule
├── .github/
│   └── workflows/
│       └── ci.yml                 # ⑤ CI: pytest on py3.9/3.11/3.12 + self-scan
├── pyproject.toml                 # ③④ identity, deps (none!), [project.scripts], tool config
├── .gitignore                     # ⑤ ignore caches, dist, venv
├── LICENSE                        # ⑤ MIT
└── README.md                      # ⑤ front page + "see it in action"
```

---

## Reading it the right way (the recipe from lesson 01.2)

**1. README.md** → "AST static analyzer that finds real bugs; here's an example run." Now you know what it does.

**2. `pyproject.toml`** → 
- `name = "codehound"`, `dependencies = []` (zero deps — pure stdlib, a flex).
- `[project.scripts] codehound = "codehound.cli:main"` → installing it gives a `codehound` terminal command that calls `main()` in `cli.py`.
- `[tool.pytest...]` → tests live in `tests/`.

**3. The package `src/codehound/`** →
- `__init__.py` → read this to see the public API: `scan_path`, `Finding`, `Check`, `ALL_CHECKS`. That's what the tool offers.
- `core.py` → the engine (parsing, the parent map from the AST lesson).
- `cli.py` → wires the engine to terminal commands.
- `checks/` → the meat: one file per rule, each a `Check` subclass. `checks/__init__.py` collects them into `ALL_CHECKS`.

**4. `tests/test_checks.py`** → for each check, a test that the bad pattern IS flagged and the good pattern is NOT. This doubles as documentation of what each rule does.

**5. `.github/workflows/ci.yml`** → runs those tests on 3 Python versions on every push, plus runs codehound on its own code.

---

## The data flow (how a run moves through the files)

```
you run:  codehound scan src
   │
cli.py  parses your command (argparse)
   │  calls
core.py scan_path() → finds .py files → ast.parse each → build parent map
   │  for each file, runs every
checks/*.py  Check.run(tree, parents, path) → returns Findings
   │  back in
cli.py  prints findings, sets exit code (non-zero if any found → CI gate)
```

You now understand a complete, real, installable Python tool — entry point, engine, plugin-style checks, tests, CI, packaging. **This structure (package + sub-package of plugins + tests + CI + pyproject) is the template for thousands of libraries.**

---

## What makes it "well-structured" (talking points)
- **Separation:** engine (`core`) vs interface (`cli`) vs rules (`checks/`). Each does one thing.
- **Extensible:** adding a rule = one file in `checks/` + one registry line + one test. Nothing else changes.
- **Zero dependencies:** only the standard library → trivial to install, nothing to break.
- **Tested + CI'd:** every rule has paired tests; CI enforces them across versions.
- **Documented:** README + `docs/ARCHITECTURE.md`.

---

## ✅ Say this in an interview

> *"codehound is laid out like a clean small library: a `src/codehound` package with the engine in `core.py`, the CLI in `cli.py`, and a `checks/` sub-package with one module per rule, collected into a registry. `pyproject.toml` declares zero dependencies and a `[project.scripts]` entry that creates the `codehound` command. Tests mirror the checks with paired good/bad cases, and CI runs them on three Python versions plus a self-scan. Adding a rule is one file plus a registry line plus a test — that separation is what makes it extensible."*

Next: how a BIG framework (agno/litellm) is organized.
