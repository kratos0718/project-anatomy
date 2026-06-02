# 03.1 — Environment Variables, `.env`, and Secrets

> Goal: fully understand `.env` — what it is, why it exists, why it's never committed. (You asked about this specifically.)

---

## The problem: code needs secrets, but code is public

Your program needs things like:
- an OpenAI API key (`sk-proj-abc123...`)
- a database password
- a Discord bot token

These are **secrets**. If you hardcode them in your `.py` files and push to GitHub, the whole world sees them — people will steal your API key and run up your bill. (This happens constantly; bots scan GitHub for leaked keys within seconds.)

Also, the *same code* needs *different* values in different places: your laptop uses a test database, production uses the real one. You don't want to edit code to switch.

**Solution: keep these values OUTSIDE the code, in the environment.**

---

## Environment variables — values that live in the OS, not your code

An **environment variable** is a name=value pair that the operating system holds, available to any program. Your code reads it instead of hardcoding:

```python
import os
api_key = os.getenv("OPENAI_API_KEY")   # read from the environment, not the file
```

You saw this exact line in agno's OpenAI tools: `self.api_key = api_key or getenv("OPENAI_API_KEY")`. The code never contains the key — it *reads* it at runtime.

Set one in your terminal:
```bash
export OPENAI_API_KEY="sk-proj-abc123"   # now os.getenv sees it
python my_app.py
```

---

## `.env` — a file to hold those variables conveniently

Typing `export` for 20 variables every time is painful. So projects use a **`.env` file**: a plain text file of `KEY=value` lines.

```bash
# .env
OPENAI_API_KEY=sk-proj-abc123
DATABASE_URL=postgresql://localhost/mydb
DISCORD_BOT_TOKEN=xyz789
DEBUG=true
```

A library (commonly **`python-dotenv`**) loads this file into the environment when the app starts:
```python
from dotenv import load_dotenv
load_dotenv()                       # reads .env into os.environ
api_key = os.getenv("OPENAI_API_KEY")
```

So `.env` is just a **convenient local container for your secrets and config**, loaded into environment variables at startup.

---

## 🔒 Why `.env` is NEVER committed to git

This is the crucial rule: **`.env` contains real secrets, so it must never go to GitHub.** Every project puts it in `.gitignore` (lesson 03.2):
```
# .gitignore
.env
```
This tells git "ignore this file — never track or upload it." Your secrets stay on your machine only.

> If you ever accidentally commit a `.env`, treat those secrets as compromised — rotate (regenerate) every key immediately. Git remembers history even after you delete the file.

---

## `.env.example` — the safe template

But if `.env` is gitignored, how does a new developer know *which* variables to set? Projects commit a **`.env.example`** (or `.env.default`, `.env.sample`) — the same keys with **fake/empty values**:

```bash
# .env.example  (this IS committed — no real secrets)
OPENAI_API_KEY=
DATABASE_URL=postgresql://localhost/mydb
DISCORD_BOT_TOKEN=your-token-here
DEBUG=false
```

A new dev copies it: `cp .env.example .env`, then fills in real values. The example documents *what's needed* without leaking *the actual secrets*. (agno's CLAUDE.md referenced `.env.default` → `.env` exactly this way.)

---

## The mental model

```
.env.example   →  committed, fake values, "here's what you need to set"
     │ copy
     ▼
.env           →  NOT committed (gitignored), YOUR real secrets
     │ load_dotenv() at startup
     ▼
os.environ     →  the running program reads via os.getenv("KEY")
```

---

## Beyond secrets: `.env` also holds config

`.env` isn't only secrets — it's also environment-specific *settings*: `DEBUG=true`, `LOG_LEVEL=info`, `PORT=8000`, `ENVIRONMENT=production`. Same idea: keep the knobs outside the code so you flip behavior without editing/redeploying code. This is part of a famous principle called the **"12-Factor App"** (store config in the environment).

---

## ✅ Say this in an interview

> *"Secrets like API keys can't live in code because the repo is public and the values differ between dev and production. So you read them from environment variables via `os.getenv`, and store them locally in a `.env` file that a loader like python-dotenv reads at startup. The `.env` is gitignored so secrets never reach GitHub; projects commit a `.env.example` with the same keys and fake values so new developers know what to set. I saw this in agno — its OpenAI tools read the key from `getenv('OPENAI_API_KEY')`, never hardcoding it."*

Next: the rest of the config files (.gitignore, YAML/TOML/JSON, settings).
