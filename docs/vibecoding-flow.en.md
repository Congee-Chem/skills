# Vibecoding Full-Stack Workflow (reading version)

> This is the human-readable version of the `/vibecoding` skill ([../vibecoding/SKILL.en.md](../vibecoding/SKILL.en.md)), writing the same flow as a readable spec.
> Core idea: **advance in the shortest verifiable steps** — every step produces something that runs, can be seen, and can be rolled back. AI is the pairing full-stack engineer, not a hands-off delegate: key architecture and security decisions are confirmed by a human.

---

## Stage 0 · Kickoff: lock the mode and the tech stack

Before any code, align on three things at once.

### 1) Collaboration mode
- **Solo**: one person + AI pairing; goal is fast, rollback-able, good enough. Light process.
- **Team**: multi-person collaboration; goal is collaborative, reviewable, traceable. Add branches / PR / CI / conventions.

### 2) Tech stack (ask 4 dimensions directly: mode / backend language / frontend / scope; confirm the rest with defaults)

> At kickoff, clarify **collaboration mode, primary backend language, frontend, scope** in one batch; confirm **database / ORM / auth / deployment** layer by layer with "recommended" defaults.

| Layer | Options | When to pick |
|---|---|---|
| **Primary backend language** | Python (FastAPI / Django) · Go (Gin·Echo) · Java (Spring Boot) · .NET (ASP.NET Core) | See the "language cheat sheet" below |
| **Frontend** | React (Next.js·recommended) · Vue (Nuxt) · SvelteKit · Astro (content site) · No frontend (pure API) | Admin/general web→Next; lightweight→SvelteKit; content site→Astro; when backend is non-JS the frontend is a standalone, independently deployed project |
| Database | **PostgreSQL (recommended)** · MySQL · SQLite (local/prototype) · MongoDB | Default Postgres; SQLite early on; don't use Mongo for relation/transaction-heavy work |
| ORM/data layer | Follow the language: Python→SQLAlchemy / Django ORM · Go→GORM / sqlc · Java→Spring Data JPA / MyBatis · .NET→EF Core | Don't hand-concatenate raw SQL strings |
| Auth | Framework-native first: Java→Spring Security · .NET→ASP.NET Identity · Python→Authlib/fastapi-users · Go→JWT libs / or hosted (Clerk/Auth0) | Don't build a password system from scratch |
| Deployment | **Docker + container platform (recommended, heavy backend)** · cloud vendor (AWS·Aliyun·Azure) · Railway·Render · Serverless (light only) | Long-running services go in containers; Serverless only for light/bursty |

**Language cheat sheet**
- **Python** — fastest prototyping/iteration; strongest AI·data·scripting ecosystem; weak at CPU-bound and high concurrency (mitigated via async). Good for MVPs, data, AI apps.
- **Go** — first choice for high-concurrency network services and cloud-native; single-binary deployment is simplest; restrained ecosystem, little boilerplate. Good for high-throughput APIs, infra, microservices.
- **Java (Spring Boot)** — large enterprise systems, strong typing, fullest ecosystem & middleware; heavy startup, lots of boilerplate, steeper curve. Good for complex business, long-evolving team systems.
- **.NET (ASP.NET Core)** — excellent performance, great C# experience, friendly with Azure/Windows; cross-platform is mature but the community skews Microsoft. Good for enterprise apps needing both performance and type safety.

### 3) Scope of this round
Be clear whether this round delivers "a complete MVP" or "adds one feature to an existing project"; write the target users and core user story in one sentence.

### 🔎 Combination sanity check (must run once after selection)

After collecting choices, **don't scaffold yet**. On a hit in the table below, explain the problem and offer an alternative, and continue only after the human decides:

| Unreasonable combo | Problem | Suggestion |
|---|---|---|
| Java / .NET + "just want to quickly validate an idea" | Heavy startup, lots of boilerplate; slows iteration in prototyping | Use Python (FastAPI) or Go for the prototype; rewrite on a heavier platform after validation |
| Go / Java / .NET long-running service + Serverless/Vercel | Unfriendly to cold starts, long connections, resident processes | Go with Docker + container platform / cloud host; leave Serverless for light functions |
| Production multi-user high concurrency + SQLite | Single write lock; concurrent writes bottleneck easily | Use PostgreSQL / MySQL; SQLite only for local/prototype/single-machine tools |
| Strong-relation, strong-transaction business + MongoDB | Cross-document transactions and joins struggle | Use PostgreSQL; use Mongo only where there's a real document/flexible-schema need |
| Pure content/showcase site + heavy backend language + relational DB | Overkill; high ops cost | Use Astro/Next static + no backend / Headless CMS |
| Backend Java/Go/.NET + frontend Next "isomorphic backend" | Isomorphic is JS-only; can't be used cross-language | Make it explicitly front/back separated: frontend does UI + calls REST/GraphQL |
| Single-person minimal project + microservice split | Ops/integration cost far exceeds the benefit | Start with a monolith; split only under clear scaling pressure |
| Chose an ORM yet hand-concatenate lots of SQL strings | Injection risk + fragmented maintenance | Standardize on ORM/query builder or parameterized queries |

> ⚠️ **Optimization point**: skipping selection and diving straight in is the most common root cause of failure. Force selection first, the sanity check, then scope alignment; write the final selection and "why" into `DECISIONS.md`.

---

## Stage 1 · Requirements clarification and slicing

- Break the idea into **user stories**: "As an X, I want Y, so that Z".
- Prioritize by value, draw the **MVP boundary**, and explicitly write down what you **won't** do this round (prevents scope creep).
- Cut the MVP into **vertical slices**: each cutting through frontend→backend→data and independently demoable; not the horizontal "finish all the backend before any frontend" layering.
- Register the slices in a task list, as the single source of truth for progress.

> ⚠️ **Optimization point**: vibecoding most easily "keeps adding features, never closes out". The MVP boundary + an explicit "won't-do list" is the brake.

---

## Stage 2 · Architecture and scaffolding

1. **Decision notes**: write a `DECISIONS.md` at the repo root, one line each ADR-style, recording selection, directory conventions, key trade-offs.
2. **Scaffolding**: bootstrap with official scaffolds, don't hand-roll config.

   **Backend**
   - Python (FastAPI): `uv init <api>` → `uv add fastapi "uvicorn[standard]" sqlalchemy alembic`; run `uv run uvicorn app.main:app --reload`
   - Python (Django): `uv init <api>` → `uv add django djangorestframework` → `django-admin startproject config .`
   - Go (Gin): `go mod init <module>` → `go get github.com/gin-gonic/gin gorm.io/gorm`; entry `main.go`
   - Java (Spring Boot): `spring init --dependencies=web,data-jpa,postgresql,security --build=gradle <api>` (or via start.spring.io)
   - .NET (ASP.NET Core): `dotnet new webapi -o <Api>` → `dotnet add package Microsoft.EntityFrameworkCore.Design`

   **Frontend** (standalone project when backend is non-JS)
   - React: `npx create-next-app@latest <web>` · Vue: `npm create nuxt@latest <web>`
   - SvelteKit: `npx sv create <web>` · Astro (content site): `npm create astro@latest <web>`

   **Database** (run locally via container, no install)
   - Postgres: `docker run -d --name pg -e POSTGRES_PASSWORD=dev -p 5432:5432 postgres:16`
   - MySQL: `docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=dev -p 3306:3306 mysql:8`

3. **Wire the first main line first**: build a minimal "frontend button → call API → read DB → show result" end-to-end skeleton, get it running locally first, then talk business.
4. Set up `.env.example`, `.gitignore`, formatting/lint (frontend Prettier+ESLint; Python Ruff; Go gofmt; Java Spotless; .NET dotnet format).

> ⚠️ **Optimization point**: wiring an end-to-end skeleton first exposes integration problems (CORS, auth, env vars, build) better than polishing a single layer. The later they're found, the more expensive.

---

## Stage 3 · Incremental implementation (the core loop)

For each vertical slice, run this small loop:

1. **Take one slice** → mark in-progress.
2. **Implement**: write the most direct working code; follow the `DECISIONS.md` conventions, reuse existing modules, match style to the surrounding code.
3. **Self-test**: actually click through it locally / call the API once, confirm behavior matches expectations.
4. **Commit**: one slice per commit; the message states "what was done, why".
5. Mark done → take the next slice. **Advance one slice at a time, don't spread half-done work in parallel.**

> ⚠️ **Optimization point**: AI tends to "generate a big blob at once that doesn't run as a whole". Force small steps: every slice runs, commits, and rolls back, for the lowest rollback cost when something breaks.

---

## Stage 4 · Verification and hardening

- **Run through the core user stories**: walk each story from Stage 1 manually.
- **Tests**: add key unit/integration tests for core business logic, covering the "change-it-and-it-breaks" paths (don't chase coverage numbers).
- **Edges and error states**: empty data, over-long input, network failure, not-logged-in, insufficient permission — at minimum don't crash.
- **Security self-check**: keys not in the repo; input validation; ORM/parameterized against injection; auth on the server side, not just hidden in the frontend.
- **Performance check**: N+1 queries, un-indexed high-frequency queries, first-screen bundle size.

> ⚠️ **Optimization point**: vibecoding often treats "it runs" as "it's done". Error states and security are the two most-skipped and most-expensive-after-launch areas.

---

## Stage 5 · Review and delivery (the two tracks)

### 🧑 Solo
- Self-review the current diff (use `code-review`), fix per its suggestions.
- Update the `README`: how to install, how to run, env var notes.
- Deploy to the chosen platform, and **smoke-test the live environment** (don't declare done after only local verification).
- Record "not yet done / known issues" in `DECISIONS.md` or issues, leaving clues for future you.

### 👥 Team
- **Branch and PR**: feature branch → open PR; the description states motivation, changes, how it was verified, risks.
- **CI**: PRs auto-run lint + test + build; merge only when green. If none, set up a minimal workflow first.
- **Code review**: run a tool first to lighten the reviewer's load; human review focuses on architecture and business correctness.
- **Convention alignment**: Conventional Commits, `CODEOWNERS`, PR template, `CONTRIBUTING.md`.
- **Environment separation**: dev / staging / prod separate environments and secrets; migration scripts (Alembic / GORM migrate / Flyway·Liquibase / EF Migrations) go into the process — never hand-edit the production DB.
- **Observability**: logging, error reporting (Sentry, etc.), basic monitoring.

> ⚠️ **Optimization point (the biggest solo→team gap)**: solo can rely on "keeping it in your head"; team must **externalize conventions into files and CI**, or collaboration grinds down in inconsistency.

---

## Principles throughout

- **Runnable first**: at any moment the working tree should be runnable.
- **Small, rollback-able steps**: prefer many small commits over one big bang.
- **Leave a decision trail**: write important trade-offs into `DECISIONS.md`, not just in conversation.
- **Align before acting**: before the next stage, restate "where we are now, what's next".
- **AI is a pairing engineer**: have the human confirm key architecture and security decisions; don't silently decide for them.

---

## Usage notes
- **Onboarding an existing project**: skip the scaffolding part, but still quickly review the current selection and fill in `DECISIONS.md`.
- **In a hurry / only a prototype**: can compress to "Solo + SQLite + hosted deployment", but don't skip the two brakes: the won't-do list and the smoke test.
