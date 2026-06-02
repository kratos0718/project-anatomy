# 07.2 — Walkthrough: A Big Framework (agno / litellm monorepos)

> Goal: not be intimidated by a 1000-file repo. Big repos use the SAME 5 buckets — just more of each, organized into sub-areas.

---

## The key idea: a big repo is small repos' patterns, repeated

A huge framework doesn't invent new concepts — it just has:
- a **bigger source package**, split into **sub-packages by feature** (models, tools, memory, db...),
- **more tests** (mirroring that structure),
- possibly **multiple packages** in one repo (a "monorepo"),
- the same config/deps/CI/meta files you already know.

If you can read codehound (lesson 07.1), you can read agno — you just navigate to the *one sub-folder* you care about and ignore the rest.

---

## "Monorepo" — multiple packages in one repository

agno keeps its main library nested:
```
agno/                              # the repo
├── libs/
│   └── agno/                      # one publishable package lives here
│       ├── agno/                  # ① the actual package (import agno)
│       │   ├── __init__.py
│       │   ├── agent/             #    sub-package: the Agent
│       │   ├── tools/             #    sub-package: tools (your B006 fixes here)
│       │   ├── models/            #    sub-package: LLM providers
│       │   ├── memory/            #    sub-package
│       │   ├── knowledge/         #    sub-package: RAG
│       │   ├── vectordb/          #    sub-package (your couchbase sleep fix)
│       │   ├── integrations/      #    sub-package (your Discord fix)
│       │   └── tracing/           #    sub-package (your exporter fix)
│       ├── tests/                 # ② mirrors the package
│       └── pyproject.toml         # ③④ this package's config
├── cookbook/                      # ⑤ runnable examples
├── scripts/                       # ⑤ format.sh, validate.sh
├── .github/                       # ⑤ CI workflows
└── README.md
```

**Why nest under `libs/agno/agno/`?** Because the repo may hold *several* packages (a monorepo). The triple nesting is: repo → `libs/<pkgdir>/` → `<importable package>/`. Don't overthink it — the actual code you import is the innermost `agno/`.

**How you navigated it:** your fixes were each in *one* sub-package:
- Discord blocking-requests → `integrations/discord/client.py`
- file-handle leak → `tools/openai.py`
- couchbase sleep → `vectordb/couchbase/couchbase.py`
- tracing exporter → `tracing/exporter.py`

You never needed to understand all of agno — just the *one file's neighborhood*. That's the skill: **find your sub-package, ignore the other 95%.**

---

## litellm — a huge flat package + a sub-application

```
litellm/                           # the repo
├── litellm/                       # ① the package (flat, but huge)
│   ├── __init__.py
│   ├── main.py                    #    core completion() function
│   ├── caching/                   #    sub-package (your cache-write fix)
│   │   └── caching_handler.py
│   ├── llms/                      #    one folder per LLM provider
│   ├── router.py                  #    load balancing
│   ├── types/                     #    type definitions
│   └── proxy/                     #    a WHOLE FastAPI app inside the package
│       ├── proxy_server.py        #       the server
│       ├── auth/                  #       authentication
│       └── db/                    #       database (Prisma)
├── tests/                         # ② mirrors it (your test: tests/litellm/caching/...)
├── enterprise/                    #    paid features, separate folder
├── requirements/                  # ④ split dependency files
├── Dockerfile                     # ④ for running the proxy server
├── Makefile                       # ⑤ make lint / make test
└── .github/workflows/             # ⑤ the CI that ran "lint" on your PR
```

Notice: litellm is *both* a library (`import litellm`) *and* ships a server (`litellm/proxy/` is a full FastAPI app with auth + db). Big projects often bundle a library + an app.

---

## How to approach ANY big repo (the 5-minute method)

1. **README** — what is it, at a high level.
2. **Find the source package** — `libs/<name>/<name>/` or `<name>/`. Skim its sub-folder names — they're the *features* (agent, tools, models...).
3. **Go straight to the ONE sub-folder relevant to your task.** Ignore everything else. You don't need the whole map, just your street.
4. **Find that file's test** — same path under `tests/`.
5. **Read `CONTRIBUTING.md` + `Makefile`** for the rules and the validate commands.

You did exactly this for every agno/litellm PR without realizing it was a repeatable method. Now it's explicit.

---

## Recognizing sub-package organization

Big source packages are split by **feature/domain**, and the folder names tell you the architecture:
- `models/` or `llms/` → LLM provider integrations
- `tools/` → agent tools/functions
- `memory/`, `knowledge/`, `vectordb/` → RAG/storage
- `integrations/` → external services (Discord, Slack)
- `api/`, `server/`, `proxy/`, `routes/` → web/API layer
- `db/`, `storage/` → persistence
- `types/`, `schemas/` → data models
- `utils/`, `core/` → shared helpers

Glance at these folder names and you've grasped the system's architecture in 30 seconds.

---

## ✅ Say this in an interview

> *"A big framework uses the same five buckets as a small one, just split into feature sub-packages — agno has agent, tools, models, memory, vectordb, integrations, tracing, each a folder. It's a monorepo so the package nests under `libs/agno/agno/`. The skill isn't understanding all of it — it's navigating to the one sub-package your change touches and ignoring the rest. For my agno fixes I went straight to `integrations/discord` or `vectordb/couchbase`, found the mirrored test path, and checked the Makefile/CONTRIBUTING for the validate commands. The folder names — models, tools, db, api — are the architecture at a glance."*

Next: a typical full-stack web app.
