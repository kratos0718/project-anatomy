# 05.2 — The `.github/` Folder (CI, templates, automation)

> Goal: understand every file in the mysterious `.github/` folder you saw on every PR.

---

## What `.github/` is

A special folder GitHub recognizes. It holds **automation and templates** for the project — not program code, pure project meta (bucket ⑤). When you opened PRs, the things that happened automatically (tests running, "Missing issue link" comments, PR templates) were configured here.

```
.github/
├── workflows/                  # CI/CD pipelines (GitHub Actions)
│   ├── ci.yml
│   ├── test.yml
│   └── release.yml
├── PULL_REQUEST_TEMPLATE.md    # auto-fills the PR description box
├── ISSUE_TEMPLATE/             # forms for filing issues
│   ├── bug_report.yml
│   └── feature_request.yml
├── CODEOWNERS                  # who must review which files
├── dependabot.yml              # auto-updates dependencies
└── FUNDING.yml                 # sponsorship links
```

---

## `workflows/*.yml` — GitHub Actions (the CI robots)

The most important part. Each `.yml` file defines a **workflow** — a set of jobs GitHub runs automatically on events (push, PR, release). This is what runs your tests and linting on every PR.

Annotated example (close to codehound's real `ci.yml`):
```yaml
name: CI                          # workflow name (shown on the PR)

on:                               # WHEN to run
  push:
    branches: [main]
  pull_request:                   # ← runs on every PR

jobs:
  test:                           # a job
    runs-on: ubuntu-latest        # on a fresh Linux machine GitHub provides
    strategy:
      matrix:
        python-version: ["3.9", "3.11", "3.12"]   # run on 3 Python versions
    steps:                        # the actual commands
      - uses: actions/checkout@v4         # 1. grab the repo code
      - uses: actions/setup-python@v5     # 2. install Python
        with:
          python-version: ${{ matrix.python-version }}
      - run: pip install -e ".[dev]"      # 3. install the package + dev deps
      - run: pytest -q                    # 4. run the tests
      - run: codehound scan src --exit-zero   # 5. self-scan
```

Key terms:
- **workflow** = a `.yml` file = a pipeline.
- **job** = a unit that runs on its own fresh machine.
- **step** = one command/action in a job.
- **runner** = the machine GitHub spins up (`ubuntu-latest`).
- **matrix** = run the same job across multiple versions (you saw "tests (3.12)" — that's one matrix cell).
- **action** = a reusable building block (`actions/checkout@v4` = "clone the repo").

When you pushed a PR and saw checks like `tests (3.12)`, `lint`, `style-check-agno` — each was a job/step from one of these workflow files. A red ✗ meant a step failed. **CI/CD = this whole automated system.** (CD = "Continuous Deployment/Delivery" — workflows that also *publish/deploy* on release, e.g. push a wheel to PyPI.)

---

## `PULL_REQUEST_TEMPLATE.md` — the pre-filled PR description

When you open a PR, GitHub pre-fills the description box with this file's content — a checklist/structure the project wants:
```markdown
## Summary
<!-- what does this PR do? -->

## Type of change
- [ ] Bug fix
- [ ] New feature

## Checklist
- [ ] I added tests
- [ ] Tests pass locally
```
You filled these out on your PRs. It's how projects enforce consistent, complete PR descriptions.

## `ISSUE_TEMPLATE/` — forms for filing issues
Structured forms (bug report, feature request) so issues have the info maintainers need. When you created bug issues (e.g. agno #8182), you filled one of these.

---

## The other files (recognize, don't sweat)
- **`CODEOWNERS`** — maps files to required reviewers ("any change to `auth/` needs @security-team").
- **`dependabot.yml`** — config for Dependabot, a bot that auto-opens PRs to update outdated dependencies (you saw `dependabot[bot]` in merge lists).
- **`FUNDING.yml`** — adds a "Sponsor" button.
- **`triage`-type workflows** — bots that auto-label PRs (agno's "Missing issue link" comment was a triage workflow).

---

## ✅ Say this in an interview

> *"The `.github/` folder holds a project's automation and templates. `workflows/*.yml` are GitHub Actions pipelines — they run jobs on fresh machines on every push or PR, like installing deps and running pytest, lint, and type-checks across multiple Python versions via a matrix. The PR and issue templates pre-fill descriptions so contributions are consistent. CODEOWNERS routes reviews, and dependabot auto-updates dependencies. Every green or red check I saw on my PRs came from a step in one of these workflow files — for instance my codehound CI runs tests on Python 3.9, 3.11, and 3.12 and even self-scans."*

Next section: Docker and deployment.
