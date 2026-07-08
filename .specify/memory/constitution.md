# MomentMan4 Project Constitution

<!--
Reusable baseline constitution for MomentMan4 projects.
Copy this file into any new Spec-Kit project (.specify/memory/constitution.md) and adjust
the [PROJECT-SPECIFIC] notes. It encodes the standard v0 + Claude Code + Next.js workflow so
every /speckit-plan and /speckit-implement run inherits these standards by default.

It covers the three kinds of projects built here: full applications, marketing websites, and
general/content websites. Principles apply to all three; where rigor should scale with the
project type, the "Right-Sized" guidance says so explicitly.
-->

## Project Types

Every project declares one of these types in its plan so the right level of rigor is applied.
The type sets expectations for testing, security, data, and state — a brochure site and a
multi-tenant app should not carry the same overhead.

- **Application** — software with real behavior: authentication, user/account data,
  persistence, APIs, non-trivial state, business logic (dashboards, SaaS, tools, portals).
  Highest rigor: security, data integrity, and automated testing are first-class.
- **Marketing website** — conversion-focused sites: landing pages, product/launch sites,
  waitlists. Content- and performance-led; light backend (forms, email capture, analytics).
- **Website (general/content)** — brochureware, portfolios, docs, blogs. Mostly static or
  CMS-driven content with minimal dynamic behavior.

A project may be a **pair** (e.g. a marketing site + its companion app) — treat each repo as
its own type.

## Core Principles

### I. Spec-Driven, Not Vibe-Driven
Every non-trivial feature starts as a written spec (`/speckit-specify`) before any code is
written. The spec describes **what** and **why** (user value, behavior, acceptance criteria)
and deliberately excludes implementation detail. Plans (`/speckit-plan`) own the **how**. Code
that has no corresponding spec/task is treated as unplanned scope and must be justified or
retro-specced. Ambiguity is resolved with `/speckit-clarify` before planning, never guessed
during implementation. This applies to all project types; applications additionally spec their
data model, auth, and key flows, not just screens.

### II. Design in v0, Engineer in Claude
UI is authored and iterated in **v0** (Vercel) and pulled into the repo; **Claude Code** owns
structure, logic, data flow, state, integration, and refactors. Neither hand-edits the other's
core concern without saying so: when Claude touches v0-generated components it preserves their
visual intent and Tailwind classes; when v0 output lands, Claude wires it to real data and the
spec's tasks rather than leaving mock content. Presentational components stay dumb; behavior
lives in hooks, server actions, services, and lib. For applications this separation is stricter:
UI never talks to the database or third-party APIs directly — it goes through a typed data/service layer.

### III. Standard Stack (NON-NEGOTIABLE unless the spec overrides)
The default stack is **Next.js (App Router) + TypeScript (strict) + Tailwind CSS + Radix/shadcn
primitives**, deployed to **Vercel**. For applications, the default backend is **Next.js server
actions / route handlers with a hosted Postgres (e.g. Supabase or Vercel Postgres) via a typed
query layer or ORM**, and authentication via a vetted provider (e.g. Supabase Auth, Auth.js) —
never hand-rolled crypto. New dependencies require a one-line justification in the plan. Prefer
platform and existing deps over new libraries (YAGNI). No `any` in committed TypeScript without
an inline reason. Server Components by default; `"use client"` only where interactivity demands it.

### IV. Data Integrity & Security (scales with project type)
Applies wherever a project stores data, authenticates users, or accepts input:
- **Validate at the boundary.** All external input (forms, params, webhooks, API bodies) is
  validated and typed at the server boundary (e.g. with a schema validator) before use.
- **Authorize every mutation and protected read.** Authentication is not authorization — every
  server action / route handler checks the caller is allowed to touch that specific resource.
  Prefer row-level security where the database supports it.
- **Least privilege & secrets.** Secrets live only in environment variables; service-role/admin
  keys never reach the client. Public (`NEXT_PUBLIC_*`) values are treated as public.
- **Safe data handling.** No secrets or PII in logs; parameterized queries only (no string-built
  SQL); explicit, documented data retention. Migrations are versioned and reviewed.

For marketing/content sites the practical floor is: validate form input, protect endpoints
against spam/abuse, and keep keys server-side. Full applications carry the whole list.

### V. Right-Sized Testing
Test depth scales with project type and risk, and the plan states the intended coverage:
- **Applications**: automated tests are required for business logic, data-layer/auth boundaries,
  and critical user flows (unit + integration; e2e for the highest-risk paths). Bugfixes add a
  regression test.
- **Marketing/content sites**: test custom logic (form handling, validation, dynamic pieces);
  visual/content correctness is verified on the Vercel preview.
Every change is verified end-to-end by exercising the actual behavior, not just by the build
passing.

### VI. Ship-Ready Increments
Every task closes in a state that builds and deploys clean: `pnpm build`, `pnpm lint`,
`pnpm typecheck`, and the project's test command (where tests exist) must pass before a task is
marked done. No committed secrets — all keys live in environment variables and `.env*` stays
gitignored; an up-to-date `.env.example` documents required variables. Broken `main` is never
acceptable; work happens on feature branches and merges only when green.

### VII. Accessible & Performant by Default
Semantic HTML, keyboard operability, focus management, and adequate color contrast are
acceptance criteria, not polish. Radix/shadcn primitives are used for interactive controls to
get a11y for free rather than reinventing them. Respect Core Web Vitals: optimize images via
`next/image`, avoid layout shift, keep client bundles lean, and lazy-load heavy/below-the-fold
work. Applications additionally guard perceived performance: loading/skeleton and error states
for async work, and no unbounded queries on hot paths.

## Technology & Structure Standards

- **Framework**: Next.js App Router. Routes/pages in `app/`, shared UI in `components/`,
  logic/helpers in `lib/`, reusable stateful logic in `hooks/`, server mutations in `actions/`.
- **Application data & backend** [PROJECT-SPECIFIC]: name the database, ORM/query layer, and
  auth provider in the plan. Keep data access behind a typed service/data layer in `lib/`
  (e.g. `lib/db`, `lib/services`) or server actions — never inline in components. Schema changes
  go through versioned migrations. External APIs are wrapped in a typed client, not called ad hoc.
- **State & data fetching**: Server Components + server actions by default; client state kept
  local and minimal; introduce a client data/query or global-state library only when the plan
  justifies it.
- **Styling**: Tailwind + `tailwind-merge`/`clsx` for conditional classes; design tokens in
  `tailwind.config.ts`; no ad-hoc inline styles when a utility class exists.
- **Components**: shadcn/Radix conventions; variants via `class-variance-authority`.
- **Errors & observability** (applications): typed error handling at boundaries, user-facing
  error/empty/loading states, and server-side error logging (no PII).
- **Environment**: all config via env vars; `.env.example` committed and current; secrets never
  committed.
- **Package manager**: pnpm (lockfile committed).
- **Deploy target**: Vercel (preview per PR, production on default branch).

## Development Workflow

1. `/speckit-constitution` — confirm/adjust these principles and declare the **project type**.
2. `/speckit-specify` — capture the feature's what/why + acceptance criteria (and data/auth
   shape for applications).
3. `/speckit-clarify` (optional) — de-risk ambiguity before planning.
4. `/speckit-plan` — architecture and stack decisions consistent with this constitution.
5. `/speckit-tasks` — ordered, independently shippable tasks.
6. `/speckit-analyze` / `/speckit-checklist` (optional) — verify cross-artifact consistency.
7. `/speckit-implement` — build against the tasks; keep build/lint/typecheck/tests green.

Quality gates before merge: build passes, lint clean, types check, required tests pass, spec
acceptance criteria met, input validated and mutations authorized (applications), no secrets
committed, a11y basics honored. UI changes are visually reviewed (Vercel preview) before merge.

## Governance

This constitution supersedes ad-hoc practice for projects that adopt it. Plans and
implementations that violate a principle must either be corrected or the principle amended
here first — deviations are documented in the plan's Complexity/Tradeoffs section, never left
implicit. The declared **project type** determines which "scales with project type" clauses
apply, but no project may silently skip a principle its type calls for. Amendments bump the
version below (semantic: MAJOR for principle removals/redefinitions, MINOR for new
principles/sections, PATCH for clarifications) and update the Last Amended date. `CLAUDE.md`
(if present) provides runtime guidance and must stay consistent with this document.

**Version**: 1.1.0 | **Ratified**: 2026-07-08 | **Last Amended**: 2026-07-08
