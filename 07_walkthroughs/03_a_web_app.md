# 07.3 — Walkthrough: A Full-Stack Web App (backend + frontend + .env)

> Goal: understand the layout of a real web application — the kind you'd build (PathForge) or see in marimo. This ties together backend, frontend, config, and deployment.

---

## The shape of a full-stack app

A web app has two halves that talk to each other:
- **Backend** — runs on a server, holds the logic + database, exposes an **API** (URLs that return data). Usually Python (FastAPI/Django) or Node.
- **Frontend** — runs in the user's browser, the UI they see and click. JavaScript (React/Next.js).

```
my-web-app/
├── backend/                       # ① Python API server
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                #    FastAPI app: defines the API routes
│   │   ├── routers/               #    URL groups (auth, users, items)
│   │   │   ├── auth.py
│   │   │   └── items.py
│   │   ├── models.py              #    database table definitions
│   │   ├── schemas.py             #    request/response shapes (Pydantic)
│   │   ├── database.py            #    DB connection setup
│   │   └── config.py              #    reads env vars into settings
│   ├── tests/                     # ②
│   ├── requirements.txt           # ④ backend deps (fastapi, sqlalchemy...)
│   ├── Dockerfile                 # ④ to containerize the backend
│   └── .env                       # ③ DB_URL, SECRET_KEY, API keys (gitignored)
│
├── frontend/                      # ① React/Next.js UI
│   ├── src/
│   │   ├── App.tsx                #    root component
│   │   ├── pages/                 #    screens/routes
│   │   ├── components/            #    reusable UI pieces
│   │   └── api/                   #    code that calls the backend API
│   ├── public/                    #    static assets (images, favicon)
│   ├── package.json               # ④ frontend deps + scripts (npm run dev/build)
│   ├── .env.local                 # ③ frontend env (e.g. API URL) gitignored
│   └── vite.config.ts             # ③ build-tool config
│
├── docker-compose.yml             # ④ run backend + frontend + db together
├── .gitignore                     # ⑤ ignores .env, node_modules, .venv, dist
└── README.md                      # ⑤
```

This is essentially your **PathForge** (FastAPI + LangChain backend + React frontend + a database).

---

## How the two halves talk: the API

1. User clicks "Load my study plan" in the **frontend**.
2. Frontend code (in `frontend/src/api/`) sends an HTTP request to a **backend** URL like `GET /api/plans`.
3. The backend **router** (`backend/app/routers/`) handles that URL, queries the database, returns JSON.
4. The frontend receives the JSON and renders it.

```
Browser (frontend)  ──HTTP request──►  Server (backend API)  ──►  Database
        ▲                                     │
        └──────────── JSON response ──────────┘
```

The **API** is the contract between them: a set of URLs (endpoints) the backend exposes and the frontend calls.

---

## Backend file roles (FastAPI-style)
- **`main.py`** — creates the app, includes the routers. The entry point.
- **`routers/`** — each file = a group of related endpoints (`auth.py` = login/signup URLs).
- **`models.py`** — database tables as Python classes (ORM, e.g. SQLAlchemy/Prisma).
- **`schemas.py`** — Pydantic models defining what requests/responses look like (validation).
- **`database.py`** — connects to the DB (URL comes from `.env`).
- **`config.py`** — reads env vars into a settings object (the bridge from lesson 03.2).

## Frontend file roles (React-style)
- **`App.tsx`** — the root; sets up routing.
- **`pages/`** — full screens (Home, Dashboard, Login).
- **`components/`** — reusable bits (Button, Card, NavBar).
- **`api/`** — functions that call the backend.
- **`package.json`** — deps + `scripts` (`npm run dev` to develop, `npm run build` to produce the production bundle).

---

## Where `.env` fits (your specific question, in context)
- **Backend `.env`** → `DATABASE_URL`, `SECRET_KEY`, `OPENAI_API_KEY`. Secret, server-side, gitignored.
- **Frontend `.env.local`** → usually just the backend's URL (`VITE_API_URL=http://localhost:8000`). ⚠️ Frontend env vars are *embedded into the browser bundle*, so they are NOT secret — never put real secrets in frontend env. Secrets live only in the backend.

This is a crucial real-world point: **secrets belong in the backend, never the frontend** (the frontend ships to the user's browser where anyone can read it).

---

## Running it locally
```bash
# backend
cd backend && python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env          # fill in real values
uvicorn app.main:app --reload # start the API on :8000

# frontend (separate terminal)
cd frontend && pnpm install
pnpm dev                      # start the UI on :5173, talking to :8000
```
Or `docker compose up` to start everything (backend + frontend + db) at once.

---

## ✅ Say this in an interview

> *"A full-stack app has a backend — a Python API server with routers, database models, Pydantic schemas, and a config that reads env vars — and a frontend, a React app with pages, components, and code that calls the backend's API over HTTP returning JSON. The two communicate through that API contract. Config splits by side: the backend `.env` holds real secrets like the database URL and API keys, while frontend env only holds non-secret values like the API URL, because anything in the frontend ships to the browser. That's exactly how my PathForge project is structured — FastAPI backend, React frontend, secrets server-side only."*

---

## 🎓 You've now read the whole repo

You can open *any* project — small library, giant framework, or full-stack app — and know what every folder and file is. That's the skill this repo set out to give you. Pair it with [oss-learning-path](https://github.com/kratos0718/oss-learning-path) (the concepts) and you can both *navigate* and *understand* real codebases.
