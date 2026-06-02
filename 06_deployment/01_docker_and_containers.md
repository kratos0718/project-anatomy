# 06.1 — Docker, Dockerfile, docker-compose

> Goal: understand `Dockerfile` and `docker-compose.yml` — the deployment files in big repos like AutoGPT and litellm.

---

## The problem Docker solves: "works on my machine"

Your app needs Python 3.11, certain libraries, an environment variable, maybe Postgres and Redis running. Getting all that set up *identically* on your laptop, a teammate's laptop, and a production server is painful and error-prone. Something's always slightly different.

**Docker** packages your app *with its entire environment* — the right Python version, all libraries, system tools, config — into a single portable unit that runs **identically everywhere**. "Build once, run anywhere."

---

## Image vs container (the key distinction)

- **Image** = a frozen, read-only template: "here's the app + everything it needs." Like a class, or a recipe, or an installer file.
- **Container** = a running instance of an image. Like an object, or the cooked dish, or the installed program. You can run many containers from one image.

```
Dockerfile  ── (build) ──►  Image  ── (run) ──►  Container(s)
 (recipe)                  (frozen template)    (running app)
```

---

## `Dockerfile` — the recipe to build an image

A text file of steps to assemble the image. Annotated:
```dockerfile
FROM python:3.11-slim          # start from a base image (Python 3.11 already installed)
WORKDIR /app                   # set the working folder inside the image
COPY requirements.txt .        # copy the deps list in
RUN pip install -r requirements.txt   # install them (baked into the image)
COPY . .                       # copy the rest of the app code in
EXPOSE 8000                    # document that the app uses port 8000
CMD ["python", "-m", "myapp"]  # the command to run when a container starts
```
Each line is a step. `docker build` runs them top to bottom to produce the image. `docker run <image>` starts a container from it.

You don't need to *write* these as a beginner — but now you can **read** one and know exactly what environment the app expects.

---

## `docker-compose.yml` — multiple services at once

Real apps aren't just one program — they're an app + database + cache + queue, all running together. **Docker Compose** defines and runs this whole stack with one command, in a YAML file:

```yaml
services:
  app:                          # your application
    build: .                    # build from the Dockerfile here
    ports:
      - "8000:8000"             # map host port 8000 → container port 8000
    env_file: .env              # load environment variables from .env
    depends_on: [db, redis]     # start db and redis first

  db:                           # a Postgres database service
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: secret

  redis:                        # a Redis cache service
    image: redis:7
```
`docker compose up` starts all three (app, db, redis) wired together. AutoGPT's backend used exactly this (`docker compose up -d` to start db, redis, rabbitmq). litellm's proxy too.

**Why you care:** when a repo has a `docker-compose.yml`, it tells you the app's *full architecture* at a glance — "oh, this needs Postgres and Redis." And `env_file: .env` shows how the secrets from lesson 03.1 get injected.

---

## The deployment picture

```
your code + Dockerfile
        │ docker build
        ▼
     Image  ──── pushed to a registry (Docker Hub / GitHub Container Registry)
        │ docker run (on a server / cloud)
        ▼
   Container running in production
```
Cloud platforms (AWS, Railway, Render, Fly.io) take your image and run it. This is "deployment" — getting your app running on a server users can reach. (Your PathForge project's Supabase backend is a managed version of this idea.)

---

## ✅ Say this in an interview

> *"Docker packages an app with its entire environment into a portable image that runs identically everywhere, solving 'works on my machine.' A Dockerfile is the recipe — base image, copy code, install deps, set the start command — and `docker build` turns it into an image, which you `docker run` as a container. docker-compose.yml defines a multi-service stack — app plus Postgres plus Redis — and starts them together, injecting secrets via env_file. When a repo has a compose file I can read the app's whole architecture from it; AutoGPT and litellm both used this for their backends."*

Next: build artifacts (dist/, wheels, what gets published).
