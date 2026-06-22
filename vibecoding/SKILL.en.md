---
name: vibecoding
description: >-
  Full-stack Vibecoding workflow: from requirements clarification → tech-stack
  selection → architecture design → scaffolding → vertical-slice incremental
  implementation → self-testing → review → deployment, driving a human + AI pair
  to ship a full-stack feature/project quickly and controllably. On invocation,
  let the user choose the tech stack and collaboration mode (solo / team), then
  proceed along the matching track. Use when the user wants to "build a full-stack
  project/feature from scratch the vibecoding way", "pair-program by a disciplined
  process", or "turn an idea into runnable, deployable code".
---

# Vibecoding Full-Stack Workflow

You (Claude) are the pairing full-stack engineer. The goal is not to write all the code in one shot, but to **advance in the shortest verifiable steps**: every step produces something that runs, can be seen, and can be rolled back. The script below drives the whole flow. **Proceed strictly stage by stage, and align with the user before entering the next stage.**

## Stage 0 · Kickoff: lock the mode and the tech stack

Before writing any code, use **one batch of `AskUserQuestion` (4 questions)** to clarify these 4 dimensions at once (don't drag it out one at a time): **① collaboration mode · ② primary backend language · ③ frontend · ④ scope of this round**. The remaining **database / ORM / auth / deployment** are then confirmed layer by layer with "recommended" defaults (if the user has no special requirement, use the recommendation — no need to open another round of questions).

1. **Collaboration mode** (Q1) — decides which track:
   - `Solo`: one person + AI pairing; goal is fast, rollback-able, good enough. Light process.
   - `Team`: multi-person collaboration; goal is collaborative, reviewable, traceable. Add branches/PR/CI/conventions.

2. **Primary backend language** (Q2) — drives the downstream ORM/auth/deployment choices. Options:
   `Python (FastAPI/Django)` / `Go (Gin·Echo)` / `Java (Spring Boot)` / `.NET (ASP.NET Core)`. When the user hesitates, advise using the "language cheat sheet" below.

3. **Frontend** (Q3) — options:
   `React (Next.js)` / `Vue (Nuxt)` / `SvelteKit` / `Astro (content/showcase site)` / `No frontend (pure API)`.
   - No strong opinion, building admin/general web → recommend **React (Next.js)**: largest ecosystem, fullest component libraries.
   - Prefer progressive/template syntax → **Vue (Nuxt)**; want ultra-light/performance → **SvelteKit**; pure content/marketing site → **Astro**; backend-only service → **No frontend**.
   - When the backend is non-JS (Go/Java/.NET), the frontend is always a **standalone project, deployed independently**, integrated via REST/GraphQL — don't use Next's isomorphic backend capabilities.

4. **Scope of this round** (Q4) — see section 3 below.

   Once these 4 are collected, quickly run through the remaining layers with "recommended" defaults (the user can change anytime):

   | Layer | Options | When to pick |
   |---|---|---|
   | Database | **PostgreSQL (recommended)** / MySQL / SQLite (local/prototype) / MongoDB | Default Postgres; SQLite in early prototyping; don't use Mongo for relation/transaction-heavy work |
   | ORM/data layer | **Follow the language**: Python→SQLAlchemy / Django ORM · Go→GORM / sqlc · Java→Spring Data JPA / MyBatis · .NET→EF Core | Don't hand-concatenate raw SQL strings |
   | Auth | **Framework-native first**: Java→Spring Security · .NET→ASP.NET Identity · Python→Authlib/fastapi-users · Go→JWT libs / or hosted (Clerk/Auth0) | Don't build a password system from scratch |
   | Deployment | **Docker + container platform (recommended, heavy backend)** / cloud vendor (AWS·Aliyun·Azure) / Railway·Render / Serverless (light only) | Long-running services go in containers; Serverless only for light/bursty |

   **Language cheat sheet** (use this to advise when the user hesitates):
   - **Python** — fastest prototyping/iteration; strongest AI·data·scripting ecosystem; weak at CPU-bound and high concurrency (mitigated via async). Good for: MVPs, data, AI apps.
   - **Go** — first choice for high-concurrency network services and cloud-native; single-binary deployment is the simplest; restrained ecosystem, little boilerplate. Good for: high-throughput APIs, infra, microservices.
   - **Java (Spring Boot)** — large enterprise systems, strong typing, fullest ecosystem & middleware; heavy startup, lots of boilerplate, steeper learning curve. Good for: complex business, long-evolving team systems.
   - **.NET (ASP.NET Core)** — excellent performance, great C# experience, friendly with Azure/Windows; cross-platform is mature but the community skews Microsoft. Good for: enterprise apps, teams needing both performance and type safety.

**Scope of this round (Q4, detail)** — is this delivering "a complete MVP" or "adding one feature to an existing project" (or "just want a quick prototype")? In one sentence, who are the target/core users and what's the core user story?

### 🔎 Combination sanity check (must run after selection; if unreasonable, proactively advise)

After collecting the choices, **don't rush to scaffold**. Run through the rules below; on a hit, explain the problem and offer an alternative, and only continue after the user decides:

| Unreasonable combo | Problem | Suggestion |
|---|---|---|
| Java / .NET + "I just want to quickly validate an idea" | Heavy startup, lots of boilerplate; slows iteration in prototyping | Use **Python (FastAPI)** or **Go** in the prototype stage; rewrite on a heavier platform after validation |
| Go / Java / .NET long-running service + **Serverless/Vercel** | Unfriendly to cold starts, long connections, resident processes | Go with **Docker + container platform / cloud host**; leave Serverless for light functions |
| Production multi-user high concurrency + **SQLite** | Single write lock; concurrent writes bottleneck easily | Use **PostgreSQL / MySQL**; SQLite only for local/prototype/single-machine tools |
| Strong-relation, strong-transaction business + **MongoDB** | Cross-document transactions and joins struggle | Use **PostgreSQL**; only use Mongo locally where there's a real document/flexible-schema need |
| Pure content/showcase site + heavy backend language + relational DB | Overkill; high ops cost | Use **Astro/Next static** + no backend / Headless CMS |
| Backend Java/Go/.NET + frontend Next "Server Actions/isomorphic backend" | Isomorphic capability is JS-only; can't be used cross-language | Make it explicitly **front/back separated**: frontend does UI + calls REST/GraphQL, backend is a standalone project |
| Single-person minimal project + microservice split | Ops/integration cost far exceeds the benefit | Start with a **monolith**; split only under clear scaling pressure |
| Chose an ORM yet hand-concatenate lots of SQL strings | Injection risk + fragmented maintenance | Standardize on ORM/query builder or parameterized queries |

> ⚠️ **Optimization point (root cause of many vibecoding failures)**: skipping selection and diving straight in, only to find mid-way that the stack is wrong. This stage forces selection first, the sanity check, then scope alignment. Record the final selection and "why" **in the project** (see `DECISIONS.md` in Stage 2) so you stop relitigating it later.

## Stage 1 · Requirements clarification and slicing

- Break the "idea" into **user stories**, each shaped like "As an X, I want Y, so that Z".
- Prioritize by value and draw the **MVP boundary**: explicitly state what you **won't** do this round (write it down to prevent scope creep ⚠️).
- Cut the MVP into **vertical slices** (each cutting through frontend→backend→data and independently demoable), rather than the horizontal "finish all the backend before any frontend" layering.
- Register the slices as a task list with `TodoWrite`, as the single source of truth for progress.

> ⚠️ **Optimization point**: vibecoding most easily falls into "keep adding features, never close out". The MVP boundary + an explicit "won't-do list" is the brake.

## Stage 2 · Architecture and scaffolding

1. **Decision notes**: write a `DECISIONS.md` at the repo root recording the tech selection, directory conventions, and key trade-offs (one line each, ADR-style). This is for future you/teammates.
2. **Scaffolding**: bootstrap with official scaffolds, don't hand-roll config. Use the command for the chosen stack:

   **Backend**
   - Python (FastAPI): `uv init <api>` → `uv add fastapi "uvicorn[standard]" sqlalchemy alembic`; run `uv run uvicorn app.main:app --reload`
   - Python (Django): `uv init <api>` → `uv add django djangorestframework` → `django-admin startproject config .`
   - Go (Gin): `go mod init <module>` → `go get github.com/gin-gonic/gin gorm.io/gorm`; entry `main.go`
   - Java (Spring Boot): `spring init --dependencies=web,data-jpa,postgresql,security --build=gradle <api>` (or generate via start.spring.io)
   - .NET (ASP.NET Core): `dotnet new webapi -o <Api>` → `dotnet add package Microsoft.EntityFrameworkCore.Design`

   **Frontend** (standalone project when backend is non-JS)
   - React: `npx create-next-app@latest <web>` · Vue: `npm create nuxt@latest <web>`
   - SvelteKit: `npx sv create <web>` · Astro (content site): `npm create astro@latest <web>`

   **Database** (run locally via container, no install)
   - Postgres: `docker run -d --name pg -e POSTGRES_PASSWORD=dev -p 5432:5432 postgres:16`
   - MySQL: `docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=dev -p 3306:3306 mysql:8`
3. **Wire the first main line first**: build a minimal end-to-end skeleton of "frontend button → call API → read DB → show result", **get it running locally first**, then talk business.
4. Set up `.env.example`, `.gitignore`, formatting/lint (frontend Prettier+ESLint; Python Ruff; Go gofmt; Java Spotless; .NET dotnet format).

> ⚠️ **Optimization point**: wiring an end-to-end skeleton first exposes integration problems (CORS, auth, env vars, build) better than polishing a single layer. Integration problems get more expensive the later they're found.

## Stage 3 · Incremental implementation (the core loop)

For **each vertical slice** in the task list, run this small loop:

1. **Take one slice** → mark in-progress in `TodoWrite`.
2. **Implement**: write the most direct working code; follow the `DECISIONS.md` conventions, reuse existing modules, match naming/style to the surrounding code.
3. **Self-test**: actually click through it locally / call the API once, confirm behavior matches expectations (use the `verify` skill when needed).
4. **Commit**: one slice per commit; the commit message states "what was done, why".
5. Mark done → take the next slice. **Advance one slice at a time, don't spread half-done work in parallel.**

> ⚠️ **Optimization point**: AI tends to "generate a big blob at once that then doesn't run as a whole". Force small steps: every slice must run, commit, and roll back. Lowest rollback cost when something breaks.

## Stage 4 · Verification and hardening

- **Run through the core user stories**: walk each story from Stage 1 manually.
- **Tests**: add key unit/integration tests for core business logic (don't chase coverage numbers; cover the "change-it-and-it-breaks" paths).
- **Edges and error states**: empty data, over-long input, network failure, not-logged-in, insufficient permission — at minimum don't crash.
- **Security self-check**: keys not in the repo; input validation; SQL injection (use ORM/parameterized); auth on the server side, not just hidden in the frontend.
- **Performance check**: obvious N+1 queries, un-indexed high-frequency queries, first-screen bundle size.

> ⚠️ **Optimization point**: vibecoding often treats "it runs" as "it's done". Error states and security are the two most-skipped and most-expensive-after-launch areas; this stage explicitly stops for them.

## Stage 5 · Review and delivery (the two tracks fork here)

### 🧑 Solo
- Self-review the current diff with the `code-review` skill, fix per its suggestions.
- Update the `README`: how to install, how to run, env var notes.
- Deploy to the chosen platform, and **smoke-test the live environment** (don't declare done after only local verification ⚠️).
- Record "not yet done / known issues" in `DECISIONS.md` or issues, leaving clues for future you.

### 👥 Team
- **Branch and PR**: feature branch → open PR; the PR description states motivation, changes, how it was verified, risks.
- **CI**: PRs auto-run lint + test + build; merge only when green. If there's no CI, set up a minimal workflow first.
- **Code review**: run the `code-review` skill first to lighten the reviewer's load; human review focuses on architecture and business correctness.
- **Convention alignment**: commit convention (Conventional Commits), `CODEOWNERS`, PR template, `CONTRIBUTING.md`.
- **Environment separation**: dev / staging / prod separate environments and secrets; migration scripts (Prisma migrate, etc.) go into the process — never hand-edit the production DB.
- **Observability**: logging, error reporting (Sentry, etc.), basic monitoring, to locate problems during multi-person collaboration.

> ⚠️ **Optimization point (the biggest solo→team gap)**: solo can rely on "keeping it in your head"; team must **externalize conventions into files and CI**, or collaboration grinds down in inconsistency.

## Principles throughout

- **Runnable first**: at any moment `main`/the working tree should be in a runnable state.
- **Small, rollback-able steps**: prefer many small commits over one big bang.
- **Leave a decision trail**: write important trade-offs into `DECISIONS.md`, not just in conversation.
- **Align before acting**: before entering the next stage, restate to the user "where we are now, what's next".
- **AI is a pairing engineer, not a hands-off delegate**: have the user confirm key architecture and security decisions; don't silently decide for them.

## Usage notes

- Onboarding an existing project: skip the scaffolding part of Stage 0, but still quickly review the current selection and fill in `DECISIONS.md`.
- User in a hurry / only wants a prototype: can compress to `Solo` + SQLite + hosted deployment, but **don't skip the two brakes: the won't-do list and the smoke test**.
